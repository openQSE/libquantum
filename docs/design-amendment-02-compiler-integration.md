# Design Amendment 02: Compiler Integration

**Status:** draft

## Purpose

This amendment defines the compiler integration boundary for libquantum. It
replaces the compiler-facing target model in the current detailed design with
one based directly on the `qhw_data` returned through QFw telemetry.
It also defines the initial Qiskit and MQT Core integrations in enough detail
to implement and test them independently.

This amendment builds on [Design Amendment 01](design-amendment-01-sdk-integration.md).
QPM discovery remains allocation-driven, `QPM.query()` remains the source of
service identity and accepted program inputs, and the detailed telemetry APIs
remain the source of the qhw device, coupling, and calibration records.

The compiler integration does not select a device or route. Libquantum
constructs eligible routes, and its direct caller selects one. When an
established SDK hides execution behind its own layers, the QFw SDK adapter is
the direct libquantum caller and performs that selection internally.

## Decisions

The implementation shall follow these decisions.

1. Libquantum retains the `qhw_data` records obtained for each
   allocated device. It does not replace those records with a reduced compiler
   target model.
2. Libquantum may extract the device identity, capacity, accepted execution
   formats, metadata completeness, and snapshot generation needed for route
   construction and operation tracking.
3. Compiler callbacks receive the retained qhw records. They do not receive a
   libquantum-generated integer coupling graph or a tuple containing only
   operation names.
4. Libquantum does not create or publish a `QB1 -> 0`-style qubit mapping.
   Qiskit and MQT Core target construction use QFw-owned qhw-to-Qiskit and
   qhw-to-MQT target helpers. Any representation required internally by an SDK
   remains an implementation detail of its helper and is not a second
   canonical device description.
5. Compiler and SDK support is discrete. Registrations list exact qualified
   versions. A minimum, maximum, range, or presumed compatibility interval is
   not accepted as evidence of support.
6. Compilation and execution remain separate. `Context.compile()` returns a
   compiler artifact without submitting it. `Context.run()` submits an artifact
   whose representation is accepted by the selected QPM route.
7. Qiskit and MQT Core are the initial real compiler integrations. Catalyst is
   deferred until its QRI submission contract, QEC resource model, and result
   lifecycle are designed and qualified.

## Architecture

```text
allocated QPM
    |
    | query(): service identity, hardware type, accepted inputs
    | telemetry: qhw device, coupling, calibration records
    v
libquantum resource discovery
    |-- retains qhw_data without reducing it
    |-- extracts route and lifecycle fields
    |-- intersects compiler outputs with QPM accepted inputs
    v
selected compiler route
    |
    | retained qhw device, coupling, and calibration records
    v
compiler plugin
    |-- Qiskit: QFw qhw-to-Target helper -> transpilation -> exchange artifact
    |-- MQT Core: QFw qhw-to-MQT helper -> compilation -> exchange artifact
    |-- Catalyst: deferred
    v
CompilerResult
    |
    | separate Context.run()
    v
selected QPM execution binding
```

The boundary has three owners.

- QFw and the QPM own device discovery and the correctness of provider
  telemetry returned as `qhw_data`.
- Libquantum owns retention, compatibility-map construction, compiler
  lifecycle, provenance association, and artifact handoff.
- Each compiler plugin owns SDK-specific target construction and compilation.

Provider-specific target logic must not spread into libquantum. Conversely, a
compiler plugin must not call QRI, resolve services, inspect reservations, or
choose a different device from the one bound to its selected route.

## Retained qhw Records

Libquantum retains the qhw records returned by QFw for each device snapshot. It
does not wrap them in another target-data type or translate their topology and
operation descriptions into a libquantum schema.

- `device` is the returned `qhw-device-v1` record.
- `coupling` is the returned `qhw-coupling-v1` record when supplied.
- `calibration` is the returned `qhw-calibration-v1` record when supplied.
- Every record retains its `schema`, device identity, metadata, extensions,
  and `raw` section.
- Missing optional records are represented by `None`. Libquantum must not
  synthesize generic coupling, operations, calibration, or provider payloads.

Libquantum treats these records as ground truth. QFw and the qhw producer own
their schema conformance and internal consistency. Libquantum does not run a
second qhw validation pass or compare overlapping qhw fields for consistency.
It retains the objects supplied by QFw and must not create a second lossy target
schema.

The following qhw coupling information must remain available to compilers.

- Original node identifiers such as `QB1`.
- Directedness and every edge expressed using those identifiers.
- Structured operations, including `name`, `native_name`, `arity`,
  `parameters`, `supported_loci`, and `properties_by_locus` when present.
- Metadata, extensions, construction sources, and raw provider information.

In particular, libquantum must not perform either of these reductions:

```python
CouplingEdge(int(source), int(target))
operations: tuple[str, ...]
```

They reject valid string node identifiers and discard the target information
needed by compilers.

## Extracted Device Summary

Libquantum still needs a small summary for work that is not compiler target
construction. Discovery extracts only the fields required for that work.

```python
@dataclass(frozen=True)
class Device:
    device_id: str
    display_name: str | None
    provider: str | None
    allocation_id: str
    execution_api_version: str
    accepted_inputs: tuple[FormatSupport, ...]
    capacity: DeviceCapacity
    metadata_complete: bool
    snapshot_generation: str
    qhw_device: Mapping[str, object]
    qhw_coupling: Mapping[str, object] | None
    qhw_calibration: Mapping[str, object] | None
```

The summary supports device identity, route construction, capacity filtering,
refresh comparison, diagnostics, and operation provenance. It is not the
source from which a compiler reconstructs topology or native operations.

`snapshot_generation` covers every retained qhw record and every extracted
selection field. A change to topology, operations, calibration identity,
accepted inputs, or usable capacity creates a new generation. Existing opened
routes retain references to the qhw records with which they were opened.

## Discovery and Retention

For each allocated QPM, initialization performs this sequence.

1. Resolve the exact service and reservation binding established by QFw.
2. Call `QPM.query()` for service identity, hardware type, device identity, and
   exact accepted program-input versions.
3. Call the existing public telemetry APIs for device, coupling, and
   calibration data.
4. Retain the returned qhw records without wrapping, interpreting, or
   validating their internal contents.
5. Extract the small set of route and lifecycle summary fields.
6. Load compiler registrations and construct the compatibility map by exact
   representation and version intersection.

Failure to obtain a required response is a discovery failure. Once QFw returns
a response, libquantum treats its contents as authoritative. Optional missing
calibration is passed to the compiler as `None`; the compiler determines
whether it can operate without it.

The complete `raw` qhw payload is retained and passed through with its record.
The compiler integration decides whether and how to use it.

## Discrete Version Registration

The existing `VersionSet` range model is replaced for compiler and SDK
qualification by exact versions.

```python
@dataclass(frozen=True)
class ExactVersions:
    values: frozenset[str]


@dataclass(frozen=True)
class ObjectTypeRegistration:
    python_type: type
    sdk: str
    sdk_versions: ExactVersions
    analyze: Callable[[object], ProgramRequirements]
```

The same rule applies to compiler input and output capabilities.

```python
@dataclass(frozen=True)
class InputCapability:
    representation: str
    versions: ExactVersions
    preference: int
    mode: InputMode


@dataclass(frozen=True)
class OutputCapability:
    representation: str
    versions: ExactVersions
    preference: int
    native: bool
    serializer_id: str
```

For example, a plugin qualified with Qiskit 2.5.2 registers `{"2.5.2"}`. It
does not register `>=2.5.2`, `2.5.*`, or a minimum and maximum. Adding support
for 2.5.3 requires its own qualification evidence and an explicit additional
value.

Version comparison is exact string equality after the owning integration has
produced its canonical version string. Libquantum does not infer semantic
compatibility from adjacent release numbers.

The representation registry may contain several registrations for the same
Python type. Eligibility retains only registrations containing the installed
SDK version exactly. Conflicting SDK identities for the same runtime type are
errors; different exact version sets are not conflicts.

## Program Import Analysis

Program analysis occurs only during `Context.import_program()`. After the
representation registry identifies the SDK object, its registered analyzer
extracts the small set of attributes needed for route filtering and stores them
in the imported `Program` as `ProgramRequirements`.

```python
program = context.import_program(sdk_object)
# program.requirements now contains logical_qubits and required_features.
```

`list_routes(program)` reads these stored requirements. It does not analyze the
SDK object again. `open_route()` and `compile()` also do not repeat analysis.
The analysis step does not use qhw data, construct an SDK target, or decide
whether a particular device is compatible.

An application-supplied requirement remains authoritative. Import analysis
fills attributes that were not supplied; it does not weaken or replace an
explicit application constraint.

## Compiler Callback Contract

The compiler callbacks remain plugin-owned and context-scoped. Their target
inputs change as follows.

```python
@dataclass(frozen=True)
class CompilerRequest:
    program: Program
    device_id: str
    snapshot_generation: str
    qhw_device: Mapping[str, object]
    qhw_coupling: Mapping[str, object] | None
    qhw_calibration: Mapping[str, object] | None
    output_representation: str
    output_version: str
```

The snapshot generation identifies the immutable opened device description.
The three qhw fields are the records returned by QFw for that device.
Libquantum passes the generation and those records directly to `compile()`.

`compile()` performs only the selected compiler route. It must not submit the
artifact. It returns the existing `CompilerResult` with:

- the compiled payload;
- its exact output representation and version;
- the original source snapshot when needed for interpretation;
- the compiler-produced target snapshot or target identity;
- layout and parameter metadata returned by the compiler;
- compiler, SDK, qhw schema, and device-generation provenance; and
- diagnostics describing any provider conversion below the compiler.

The plugin returns the artifact described by its compiler result. Libquantum
treats that result as authoritative and passes its payload through the selected
route without inspecting or validating compiler-owned contents.

## Route Construction

Compatibility-map construction is mechanical. For every compiler output and
QPM input with the same representation and at least one identical exact
version, libquantum creates the corresponding route definition and associates
the returned qhw records with it. Libquantum does not judge the correctness
of either capability report or inspect qhw contents while building the map.

`list_routes(program)` filters that map using the program representation and
requirements already supplied to libquantum. It compares logical-qubit demand
with the device summary and required features with the compiler's declared
capabilities. It does not invoke a compiler compatibility callback or build an
SDK target while filtering routes.

Libquantum returns a compiler route only when all of these conditions hold.

1. The compiler plugin initialized successfully and reports an exact supported
   compiler version.
2. The program's exact SDK or serialized representation version matches one of
   the plugin's input capabilities.
3. One compiler output representation and exact version matches an input
   advertised by the selected QPM.
4. The required qhw responses were returned by QFw.
5. The program requirements fit the compiler capabilities and extracted device
   summary.

Compiler capabilities and QPM inputs are authoritative. A returned route is
therefore compatible according to the information they declared. The selected
compiler remains responsible for compiling the actual program; a compiler
failure is reported by `compile()` and does not trigger route reselection.

Libquantum enumerates all eligible routes. It does not rank them automatically
or select one for the caller. A QFw SDK adapter may apply its own deterministic
SDK-facing policy because that adapter is the direct libquantum application.

## Qiskit Compiler Integration

The Qiskit integration is an independently installed compiler plugin. It is
separate from `QFwBackendV2`; the backend uses the plugin through libquantum's
ordinary route and compilation APIs.

### Registration

The plugin registers:

- the exact `qiskit.circuit.QuantumCircuit` Python type;
- every individually qualified Qiskit version;
- the Qiskit object representation used by libquantum;
- each exchange representation and exact version it can produce for QPM
  submission; and
- a serializer identifier for each output.

Registration must import Qiskit only inside the plugin boundary. Libquantum's
core package remains importable when Qiskit is absent. If the installed Qiskit
version is not explicitly registered, the plugin reports unavailable rather
than guessing compatibility.

### Target construction

The plugin passes the retained qhw device, coupling, and calibration records
to a public QFw qhw-to-Qiskit target helper. The helper returns a Qiskit
`Target` suitable for the exact qualified Qiskit version.

The plugin does not:

- reduce qhw operations to names;
- convert qhw node identifiers into a libquantum coupling graph;
- create its own persistent node-index mapping;
- infer all-to-all connectivity;
- invent durations, error rates, or instruction properties; or
- construct `QFwBackend` or `QFwBackendV2` to obtain a target.

If QFw's current target builder is private or coupled to backend construction,
QFw must expose a reusable public helper accepting qhw records. The helper owns
the mechanics required by Qiskit's target API. The qhw records remain the
canonical target description, and the returned Qiskit object is scoped to the
compilation call unless the plugin caches it under the full device-generation
and exact-version key.

The target cache key is:

```text
(compiler_id, compiler_version, qiskit_version, device_id,
 snapshot_generation)
```

A refresh never mutates a cached target for an older opened route.

The qhw-to-Qiskit helper is a transitional integration boundary, not a
permanent QFw abstraction. Once qhw is formalized as a standard target contract
and Qiskit compiler integration can consume that standard contract directly,
the helper can be removed. Libquantum's retained qhw data and compiler callback
boundary remain unchanged through that migration.

### Import analysis

When `Context.import_program()` receives a `QuantumCircuit`, the Qiskit object
registration's analyzer obtains logical-qubit count and relevant program
features without changing the circuit. It stores those values in
`Program.requirements`. Later route queries consume the stored values; they do
not invoke the analyzer again. Qiskit target construction does not occur until
`compile()`.

### Compilation

`compile()` performs this sequence.

1. Build or retrieve the Qiskit `Target` from the qhw records retained by the
   opened route.
2. Copy or otherwise preserve the application's source circuit.
3. Invoke the qualified Qiskit transpilation path using that target and the
   plugin's deployment-owned configuration.
4. Preserve the compiler-produced layout, parameter identities, measurement
   interpretation, and relevant metadata.
5. Serialize to the exact negotiated output representation and version.
6. Return a `CompilerResult` without calling QRI.

The plugin must not use an unseeded nondeterministic default when compilation
reproducibility is part of the qualified path. Deployment-selected compilation
configuration is captured in provenance. Application-facing compiler options
remain outside the current scope unless added by a later design.

### Qiskit tests

Tests use representative qhw records with named nodes, non-all-to-all
connectivity, distinct one- and two-qubit supported loci, and optional
calibration. They prove that:

- the QFw helper receives the original qhw records;
- unsupported SDK versions are rejected;
- unsupported gates or loci fail compilation clearly;
- the source circuit is not mutated;
- layout and parameter metadata survive compilation;
- the output representation/version exactly matches the selected route; and
- no QRI submission occurs during `compile()`.

## MQT Core Compiler Integration

The MQT Core integration is a separate independently installed compiler
plugin. It uses the same libquantum registration and callback contracts rather
than being selected through a special Qiskit-backend option.

### Registration

The plugin registers every exact qualified MQT Core version and every exact
SDK-object or serialized representation it can ingest. It advertises only
output artifacts that have a defined serializer and a corresponding QPM input
representation.

If the supported application input is a Qiskit `QuantumCircuit`, that Python
type registration is explicit and names the exact qualified Qiskit versions in
addition to the exact MQT Core version reported by `CompilerInfo`. If MQT Core
also accepts its own circuit type, that type receives a separate registration.

### Target construction

The plugin passes the retained qhw device, coupling, and calibration records to
a public QFw qhw-to-MQT target helper. The helper returns an MQT target suitable
for the exact qualified MQT Core version. The plugin does not call QRI and does
not ask the application for connectivity, native operations, or a device type.

The construction must use, as applicable:

- qhw node and edge topology;
- operation arity and supported loci;
- native operation names and parameter descriptions;
- calibration or operation properties used by the selected MQT objective; and
- provider-specific raw fields when required by the qualified target helper.

No generic architecture may be substituted when required information is
missing. If target construction cannot use the returned qhw data, `compile()`
reports the target-helper failure.

The qhw-to-MQT helper follows the same boundary as the qhw-to-Qiskit helper. It
accepts qhw records directly, performs only target conversion, and does not
resolve reservations, select routes, compile programs, or submit work. Any
MQT-specific node representation remains internal to the helper and is never
stored as libquantum's canonical device description.

This helper is also transitional. Once qhw is formalized as a standard target
contract and MQT Core integration can consume that contract directly, the
qhw-to-MQT helper can be removed without changing libquantum discovery, route
construction, or compiler callback contracts.

### Import analysis and compilation

If MQT registers its own SDK object type, that object registration analyzes it
during `Context.import_program()` and stores its logical-qubit demand and
required features in `Program.requirements`. For a Qiskit-origin program, MQT
uses requirements stored during `Context.import_program()` and never analyzes
the `QuantumCircuit` again during route listing or compilation. When a shared
qualified Qiskit object registration is loaded, that registration supplies the
analyzer. In the separate MQT environment, the MQT plugin supplies an
equivalent non-mutating analyzer for its exact converted-input pair. MQT target
construction does not occur until `compile()`.

`compile()`:

1. builds or retrieves the MQT target through the QFw helper using the bound
   qhw snapshot;
2. converts the registered input into the qualified MQT representation when
   conversion is part of the declared input mode;
3. invokes the qualified MQT compilation path;
4. captures placement/layout, parameters, and objective/configuration data;
5. serializes to the negotiated QPM input representation; and
6. returns `CompilerResult` without submission.

Any Qiskit-to-MQT conversion is part of the plugin's declared converted-input
path. It is visible in provenance and cannot be described as native input.

The initial MQT qualification uses MQT Core 4.1.1 with Qiskit 2.5.2, matching
the separate `mqt-cc` environment in QFw-SLURM-Cluster. MQT target compilation
emits OpenQASM 3.1. The plugin reimports that artifact through MQT's public
`QCProgram` API, converts it through MQT's public Qiskit exporter, validates
that any generated classical initialization is overwritten by terminal
measurements before removing it, and serializes the resulting static circuit
as OpenQASM 2.0 for the qualified QPM contract. These conversions are recorded
in provenance. Programs with control flow, classical reads before overwrite,
or unbound parameters are rejected instead of being downgraded.

QFw's Qiskit environment pins Qiskit 2.2.3, while MQT Core's qualified Qiskit
frontend pins 2.5.2. The Qiskit and MQT compiler packages therefore occupy
separate clean environments, as the reference cluster already does. They may
share libquantum and QFw helper wheel revisions, qhw fixtures, and QPM
contracts, but are not required or claimed to be co-installable in one Python
environment.

### MQT tests

Tests use the same representative qhw records as the Qiskit integration so
that target behavior can be compared without changing the device description.
They prove exact-version rejection, named-node handling through the qualified
helper, topology and native-operation enforcement, preservation of layout and
parameters, exact output serialization, and absence of QRI calls during
compilation.

## Catalyst Deferral

Catalyst support is not part of the initial compiler implementation. A plugin
must not be registered merely because Catalyst can compile locally or because
QFw can reserve a Backline executor.

Before Catalyst is added, a later design amendment must define and validate:

- the qhw or other versioned records describing QEC, decoder, controller,
  coprocessor, and classical-resource placement;
- the exact Catalyst input and output versions;
- the compiled artifact accepted through QRI;
- the boundary between Catalyst lowering and any QPM/provider compilation;
- QRI status, cancellation, completion, and qhw-result behavior for that
  artifact; and
- at least one real end-to-end Catalyst-to-QRI route.

Until then, libquantum reports no Catalyst compiler route. Backline's direct
executor flow is not counted as a libquantum compiler integration.

## Artifact Submission

`Context.compile()` returns a `CompilerResult`. Submission is a later explicit
operation.

```python
compiled = context.compile(route, program)
op_id = context.run(route, compiled, completion_queue)
```

Libquantum treats `CompilerResult` as the compiler's authoritative output. It
does not inspect the payload, revalidate compiler metadata, or compare the
result with the compiler's advertised capabilities. It submits the returned
payload through the QRI binding already stored on the selected route. The
compiler plugin never sees that binding. QRI's returned `cid` remains the
libquantum operation ID.

If QFw performs an additional provider compilation or serialization stage,
that stage must be declared by the QPM input contract and recorded in result or
execution provenance. It must not cause libquantum to mislabel an exchange
artifact as device-native.

## Refresh, Concurrency, and Lifecycle

Compiler plugins initialize once per libquantum context and finalize after all
operations and opened routes using their state have been released.

Compiler target caches are context-owned and keyed by immutable snapshot
generation and exact software versions. Cache entries may be shared by
concurrent compilations only when the underlying SDK target object is safe for
that use. Compiler-owned mutable state must be protected inside the plugin;
libquantum does not serialize all compilation globally.

Resource refresh creates a new device snapshot with newly returned qhw records.
It does not mutate an opened route or replace the qhw records of an in-progress
compilation. New route queries use the new generation. An opened route
continues using the bindings and qhw records captured when it was opened.

Plugin finalization is idempotent and safe after partial initialization.
Failures in one optional compiler plugin do not corrupt retained qhw data or
another plugin's state. Whether initialization continues without that plugin
is determined by the deployment's enabled-compiler configuration.

## Failure Model

Compiler integration failures use structured stage and reason information.
At minimum, the implementation distinguishes:

- unsupported exact compiler or SDK version;
- failure to obtain a required qhw response;
- incomplete target metadata;
- target-helper failure;
- unsupported program feature, operation, or locus;
- unsupported compiler output representation/version;
- compilation failure;
- serializer failure; and
- QRI submission failure.

Diagnostics include compiler ID/version, device ID, snapshot generation, qhw
schema versions, and the rejected representation/version. They must not expose
credentials, reservation internals, or provider secrets contained in raw data.

## Required QFw Work

QFw must provide or expose these reusable mechanisms without requiring backend
construction.

1. `QPM.query()` returns service/device identity, hardware type, and exact
   accepted program inputs as defined by Design Amendment 01.
2. Existing telemetry calls return authoritative qhw device, coupling, and
   calibration records for the allocated device.
3. A public qhw-to-Qiskit `Target` helper accepts those records directly and
   supports the qualified Qiskit versions.
4. A public qhw-to-MQT target helper accepts those records directly and
   supports the qualified MQT Core versions.
5. QPM submission documents the exact exchange artifacts and versions it
   accepts, including the preservation of layout and parameter data.

These helpers are conversion utilities, not execution backends. They do not
resolve reservations, submit work, or choose routes.

They are temporary bridges for SDKs whose current target APIs do not consume
qhw directly. Standardization of qhw, together with direct compiler support for
the standard contract, removes the need for both QFw-specific conversion
helpers. Implementations must therefore keep helper-specific types and policy
out of libquantum's public and retained device models.

## Required libquantum Work

The implementation must:

1. replace range-based compiler and SDK qualification with exact version sets;
2. retain and generation-track the qhw records without validating or
   reducing their contents;
3. remove compiler dependence on integer `CouplingEdge` and operation-name-only
   device data;
4. pass the retained qhw device, coupling, and calibration records directly to
   the compilation callback;
5. submit compiler results without inspecting compiler-owned contents;
6. add the independently installable Qiskit plugin;
7. add the independently installable MQT Core plugin;
8. keep Catalyst unregistered until its later design gate is satisfied; and
9. supply dummy qhw records and dummy compilers that exercise the same callback
   and qhw pass-through paths without installing real compiler SDKs.

## Implementation Boundaries

The work should be delivered in independently testable commit passes.

1. **qhw retention:** unchanged qhw records, generation, and the
   extracted device summary.
2. **Compiler contract revision:** exact version sets and direct qhw fields in
   compilation requests, with dummy compiler coverage.
3. **Artifact handoff:** compiler-result pass-through and QRI submission through
   the selected route.
4. **QFw target helper:** backend-independent qhw-to-Qiskit target construction.
5. **Qiskit plugin:** registration, analysis, compilation, serialization, and
   isolated tests.
6. **QFw MQT target helper:** backend-independent qhw-to-MQT target
   construction with isolated target tests.
7. **MQT plugin:** registration, analysis, compilation, serialization, and
   isolated tests.
8. **Integrated qualification:** dummy end-to-end tests first, followed by
   environment-gated tests against the pinned real SDK and QFw versions.

Each pass updates the implementation plan before code changes begin. During
implementation, the agent repeatedly checks the active plan and marks each
completed checklist item as soon as its test passes. It must not infer work
beyond the selected pass or silently substitute a simpler target model.

## Test Strategy

Testing has three layers.

### Contract tests without compiler SDKs

Repository-owned qhw fixtures and dummy compiler plugins verify:

- complete qhw records reach callbacks unchanged;
- string node identifiers and structured operations survive discovery;
- exact version matching rejects unlisted adjacent versions;
- refresh creates a new generation without mutating old routes;
- compiler results pass through without payload inspection; and
- compilation never invokes QRI.

### Compiler-specific isolated tests

Qiskit and MQT tests are dependency-gated and pin exact qualified versions.
They build targets and compile representative programs without QFw services,
credentials, reservations, network access, or hardware.

Each compiler phase includes a runnable test document naming prerequisites,
fixture records, command, and expected result.

### QFw integration qualification

Deployment tests verify that real QPM telemetry reaches libquantum as qhw data,
that the same retained records can construct compiler targets, and that each
selected compiler artifact can be submitted through a QPM advertising its
exact representation/version. Qualification records the QFw revision,
compiler version, SDK version, qhw schema version, provider/device identity,
and result provenance.

## Acceptance Criteria

This amendment is implemented when all of the following are true.

- Libquantum retains the qhw device, coupling, and calibration records returned
  for every compiler-capable device snapshot without validating or reducing
  their contents.
- Compiler callbacks receive those records without an integer-edge or
  operation-name-only reduction.
- Libquantum creates no canonical or public qubit-index mapping.
- Device identity, capacity, accepted formats, completeness, and generation
  remain available for route and lifecycle operations.
- SDK and compiler support uses exact explicitly qualified versions only.
- Qiskit target construction is reusable without constructing a QFw backend.
- MQT target construction is reusable without constructing a QFw backend or
  another execution adapter.
- Qiskit compilation returns a route-valid exchange artifact without
  submission.
- MQT target construction and compilation use the same retained qhw records
  and return a route-valid exchange artifact without submission.
- Dummy plugins prove libquantum behavior without real compiler installations.
- Real integration tests are gated on their exact pinned environments.
- Catalyst remains unavailable until its separate design and qualification
  requirements are satisfied.

## Non-Goals

This amendment does not add automatic route ranking, application-configurable
compiler pass pipelines, runtime plugin installation, compiler fallback after
route selection, cross-generation artifact reuse, or Catalyst execution. It
does not replace qhw-data schemas, add a second device-target schema, or move
provider telemetry discovery into compiler plugins.
