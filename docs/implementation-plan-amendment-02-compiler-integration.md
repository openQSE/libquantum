# Compiler Integration Amendment Implementation Plan

**Status:** draft

## A02-GUIDANCE. Instructions for the Implementation Agent

Before coding, read the complete amendment, the current unit, and every linked dependency. Keep this plan open throughout implementation. Compiler-plan work that is a prerequisite of the SDK Integration Amendment Implementation Plan takes precedence over the dependent SDK unit, regardless of document number. Work through prerequisite units and checklist items in order, complete their gates, then return to the dependent SDK unit. Check a box only after its implementation and stated evidence are complete.

At the end of every unit, update this JSONL source, render its Markdown companion with the ElectroBoy artifact renderer, and include the checklist update in the same commit pass. One unit is one reviewable commit pass. Do not start the next unit while the current unit has unchecked implementation work.

Do not silently skip, combine, reinterpret, or expand tasks. If code, another plan, or an upstream contract conflicts with the amendment, stop at the affected checkbox and update the governing design or plan before continuing.

## A02-VERIFICATION. Phase Verification Contract

Every phase ends with a runnable gate and companion test procedure in the owning repository. The procedure records linked design sections, prerequisites, fixtures, exact commands, assertions, expected output, cleanup, and retained evidence.

Isolated gates require no network, credentials, allocation, or hardware. Deployment gates are explicitly marked and a skip is not qualification evidence. Fix every defect discovered by a gate before checking the phase complete.

## A02-SCOPE. Scope, Ownership, and Naming

This plan implements [Design Amendment 02: Compiler Integration](design-amendment-02-compiler-integration.md).

Repository and package naming:

- `/home/a2e/ORNL/Quantum/openQSE/libquantum-compiler-qiskit` is the distribution/repository; `libquantum_compiler_qiskit` is its Python package; `qiskit` is its compiler ID.
- `/home/a2e/ORNL/Quantum/openQSE/libquantum-compiler-mqt` is the distribution/repository; `libquantum_compiler_mqt` is its Python package; `mqt` is its compiler ID.
- Each package uses separate `plugin.py`, `analysis.py`, `compiler.py`, and `serialization.py` modules, a packaged `compiler.json`, independent build metadata, tests, CI, README, and license.
- Each package exposes `plugin_path()` so deployment code can add its installed manifest directory to `InitConfig.compiler_paths`; libquantum core imports neither optional package.
- QFw owns the transitional qhw-to-Qiskit and qhw-to-MQT helpers.

[SDK Integration Amendment Implementation Plan](implementation-plan-amendment-01-sdk-integration.md) owns QPM query population, `QFwBackendV2`, and `QFwDevice`.

## Phase 1

### A02-PH1-C1. Amended Core Contracts

Design sections: [Retained qhw Records](design-amendment-02-compiler-integration.md#retained-qhw-records), [Extracted Device Summary](design-amendment-02-compiler-integration.md#extracted-device-summary), [Discrete Version Registration](design-amendment-02-compiler-integration.md#discrete-version-registration), [Program Import Analysis](design-amendment-02-compiler-integration.md#program-import-analysis), and [Compiler Callback Contract](design-amendment-02-compiler-integration.md#compiler-callback-contract).

Current implementation affected: `libquantum/_records.py`, `libquantum/compiler.py`, `libquantum/__init__.py`, and their public API/data-model tests.

Checklist:

- [x] Keep the existing `VersionSet` as the exact discrete-version value; document and test that it performs membership only and introduce no duplicate `ExactVersions`, range, minimum, maximum, wildcard, or adjacency behavior.
- [x] Replace `Device.operations` and `Device.coupling_graph` with direct references to the returned qhw device, coupling, and calibration records while retaining only device identity, accepted inputs, capacity, completeness, and generation fields required by libquantum.
- [x] Remove `CouplingEdge` and `CouplingGraph` from the compiler path and public exports; update the API gate so no implementation agent reconstructs a second canonical topology model.
- [x] Add the import analyzer callable to `ObjectTypeRegistration`; define it to return `ProgramRequirements` from the SDK object without receiving qhw data or selecting a route.
- [x] Remove `ProgramAnalysis`, `CompatibilityRequest`, and `CompatibilityReport` from the compiler-facing contract and remove `check_compatibility()` from `CompilerCallbacks` and registration validation.
- [x] Change `CompilerRequest` to carry the exact qhw device, coupling, and calibration records separately, alongside the program and negotiated output representation/version.
- [x] Keep the core package free of Qiskit, MQT Core, PennyLane, QFw backend, and compiler-package imports.
- [x] Update type exports, docstrings, fixtures, and signature assertions together so no obsolete contract remains discoverable.

Commit boundary: revised libquantum public/compiler contract types only; no discovery or route behavior change yet.

### A02-PH1-C2. Amended Contract Phase Gate

Design sections: [Decisions](design-amendment-02-compiler-integration.md#decisions) and [Compiler Callback Contract](design-amendment-02-compiler-integration.md#compiler-callback-contract).

Checklist:

- [x] Create `tests/phase_tests/test_phase_13_amended_contracts.py`.
- [x] Prove qhw records with string nodes, structured operations, metadata, extensions, and raw payloads retain object identity through the new device and compiler-request records.
- [x] Prove the public package no longer exports integer coupling records or compatibility callback records.
- [x] Prove exact discrete versions accept only explicitly listed values.
- [x] Prove the compiler protocol requires import analysis and compilation but has no target compatibility callback.
- [x] Prove core import succeeds without any SDK, compiler distribution, or QFw installation.
- [x] Create `docs/phase-tests/phase-13-amended-contracts.md` and run `python3 -m pytest -q tests/phase_tests/test_phase_13_amended_contracts.py`.

Commit boundary: Phase 13 executable amended contract and migration evidence.

## Phase 2

### A02-PH2-C1. Query-Driven Discovery and qhw Retention

Design sections: [Allocation-Scoped QPM Discovery](design-amendment-01-sdk-integration.md#allocation-scoped-qpm-discovery), [libquantum Initialization](design-amendment-01-sdk-integration.md#libquantum-initialization), and [Discovery and Retention](design-amendment-02-compiler-integration.md#discovery-and-retention).

Current implementation affected: `libquantum/_qfw_runtime.py`, `libquantum/_qri.py`, `libquantum/_records.py`, `tests/fixtures/dummy_qri.py`, and discovery tests.

Checklist:

- [x] Keep `QFW_RESERVATIONS` parsing and exact reserved-service resolution in the QFw runtime bridge; require no provider, hardware type, or device selector from the application.
- [x] Replace `_qri.py`'s `get_program_input_capabilities()` call with one `QPM.query()` call and read exact `program_inputs` from its existing properties mapping.
- [x] Continue calling the public qhw device, coupling, calibration, and capacity telemetry operations after query discovery.
- [x] Retain the returned qhw device, coupling, and calibration records unchanged; do not call `CouplingEdge(int(...))`, reduce operations to names, add a qhw validator, or compare overlapping qhw fields.
- [x] Extract only the summary fields needed for device identity, capacity filtering, accepted-input mapping, diagnostics, and generation.
- [x] Include the returned qhw records and extracted selection fields in snapshot generation without rewriting the records.
- [x] Preserve the private execution/telemetry binding, reservation ID, and service ID used by opened routes.
- [x] Update dummy QPM query and qhw telemetry fixtures to mirror `DEFwServiceInfo.properties` and representative qhw records with string node IDs.
- [x] Delete the old separate-capability and integer-coupling fixture modes; retain failures only for missing required calls or unusable required summary fields.

Commit boundary: libquantum query-driven discovery and unchanged qhw retention.

### A02-PH2-C2. Query and qhw Discovery Phase Gate

Design sections: [Allocation-Scoped QPM Discovery](design-amendment-01-sdk-integration.md#allocation-scoped-qpm-discovery) and [Retained qhw Records](design-amendment-02-compiler-integration.md#retained-qhw-records).

Checklist:

- [x] Create `tests/phase_tests/test_phase_14_query_qhw_discovery.py`.
- [x] Prove one and multiple reserved QPMs are discovered solely from allocation bindings and `query()`.
- [x] Prove compiler output routes use only exact `program_inputs` returned by query.
- [x] Prove qhw device, coupling, calibration, metadata, extensions, raw payloads, and string node identifiers reach the device snapshot unchanged.
- [x] Prove refresh creates a new generation while an opened older route retains its original qhw objects and bindings.
- [x] Prove absence of a device-type environment variable has no effect and the old capability RPC is never called.
- [x] Re-run the legacy discovery, context lifecycle, mapping, and isolated workflow phase tests after updating their fixtures.
- [x] Create `docs/phase-tests/phase-14-query-qhw-discovery.md` and run `python3 -m pytest -q tests/phase_tests/test_phase_14_query_qhw_discovery.py`.

Commit boundary: Phase 14 runnable query/qhw discovery proof.

## Phase 3

### A02-PH3-C1. Import-Time Program Analysis

Design sections: [Program Import Analysis](design-amendment-02-compiler-integration.md#program-import-analysis) and [Route Construction](design-amendment-02-compiler-integration.md#route-construction).

Current implementation affected: `libquantum/_compiler_loader.py`, `libquantum/_handles.py`, compiler fixtures, and import tests.

Checklist:

- [x] Extend the representation registry so an exact object-type/version match returns its registered analyzer together with the representation identity.
- [x] Invoke the analyzer once inside `Context.import_program()` and store its returned logical-qubit count and required features in `Program.requirements`.
- [x] Merge explicit caller requirements by filling only missing values and unioning required features; never weaken caller constraints.
- [x] For a Qiskit-origin object, run the Qiskit object analyzer once even when both Qiskit and MQT compilers accept that representation.
- [x] Keep serialized and opaque import behavior unchanged unless their registration explicitly supplies requirements.
- [x] Remove all compiler analysis calls and analysis caches from `_selection.py`; `list_routes()`, `open_route()`, and `compile()` must consume stored requirements only.
- [x] Record analyzer calls in dummy fixtures and prove repeated route queries do not repeat them.

Commit boundary: one-time import analysis and stored program requirements.

### A02-PH3-C2. Mechanical Route Mapping and Filtering

Design section: [Route Construction](design-amendment-02-compiler-integration.md#route-construction).

Current implementation affected: `libquantum/_compatibility.py`, `libquantum/_selection.py`, and mapping/selection tests.

Checklist:

- [x] Preserve mechanical compatibility-map construction as the exact cross-product of compiler inputs/outputs and QPM inputs sharing representation and explicit version.
- [x] Associate the retained qhw records with the device referenced by each route without inspecting qhw topology, operations, calibration, or raw fields.
- [x] Remove `check_compatibility()` dispatch, compatibility reports, target-completeness callbacks, and callback-specific rejection stages from route listing.
- [x] Filter `list_routes(program)` only by stored program representation/version, stored logical-qubit demand, declared compiler features, extracted usable capacity, and explicit `RouteConstraints`.
- [x] Return every eligible route in deterministic route-ID order and perform no target construction, compilation, ranking, or fallback.
- [x] Keep `list_routes()` without a program available for SDK adapters that represent one structural backend route before receiving circuits.
- [x] Update explanation output so it reports only filters libquantum actually applies.

Commit boundary: deterministic callback-free route map and query.

### A02-PH3-C3. Compiler Dispatch and Artifact Handoff

Design sections: [Compiler Callback Contract](design-amendment-02-compiler-integration.md#compiler-callback-contract), [Artifact Submission](design-amendment-02-compiler-integration.md#artifact-submission), and [Refresh, Concurrency, and Lifecycle](design-amendment-02-compiler-integration.md#refresh-concurrency-and-lifecycle).

Current implementation affected: `libquantum/_handles.py`, `libquantum/compiler.py`, dummy compiler plugins, and compilation tests.

Checklist:

- [x] Bind compiler callbacks/state, qhw record references, and QRI dispatch once when opening a route.
- [x] Build `CompilerRequest` with the exact retained qhw device, coupling, and calibration objects rather than the reduced `Device` target fields.
- [x] Call only the selected compiler's `compile()` callback and return its `CompilerResult` as authoritative without inspecting payload or compiler-owned metadata.
- [x] Keep compilation and submission separate; `compile()` must not call QRI and `run()` must not call the compiler.
- [x] Submit `CompilerResult.payload` through the already selected QRI binding without comparing it again with advertised compiler capabilities.
- [x] Keep opened routes bound to their original qhw records across refresh.
- [x] Update dummy compilers to record direct qhw objects, remove analysis/compatibility methods, and return deterministic authoritative results.
- [x] Retain compiler exception translation as a compilation-stage failure and QRI exception translation as a submission-stage failure.

Commit boundary: direct qhw compiler dispatch and authoritative artifact handoff.

### A02-PH3-C4. Core Compiler Flow Phase Gate

Design sections: [Program Import Analysis](design-amendment-02-compiler-integration.md#program-import-analysis), [Route Construction](design-amendment-02-compiler-integration.md#route-construction), and [Artifact Submission](design-amendment-02-compiler-integration.md#artifact-submission).

Checklist:

- [x] Create `tests/phase_tests/test_phase_15_core_compiler_flow.py`.
- [x] Prove import analysis runs once, route listing runs no callbacks, and compilation receives the identical qhw objects retained from discovery.
- [x] Prove a 1,000-qubit program returns no route when usable capacity is smaller without invoking a compiler callback.
- [x] Prove exact output/input matching creates the expected route set and unreported versions create no route.
- [x] Prove compiler results pass unchanged to the caller and their payloads pass to QRI without compiler-result validation.
- [x] Prove refresh does not replace qhw objects bound to an opened route.
- [x] Re-run phases 3, 6, 7, 8, 10, and 11 after migrating their fixtures and assertions away from removed callbacks.
- [x] Create `docs/phase-tests/phase-15-core-compiler-flow.md` and run `python3 -m pytest -q tests/phase_tests/test_phase_15_core_compiler_flow.py`.

Commit boundary: Phase 15 complete isolated amended libquantum flow.

## Phase 4

### A02-PH4-C1. QFw qhw-to-Qiskit Target Helper

Design sections: [Qiskit Target Construction](design-amendment-02-compiler-integration.md#target-construction) and [Required QFw Work](design-amendment-02-compiler-integration.md#required-qfw-work).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/QFw`.

Checklist:

- [x] Extract or create a public backend-independent qhw-to-Qiskit helper in a reusable QFw module; do not construct `QFwBackend` or `QFwBackendV2`.
- [x] Accept qhw device, coupling, and calibration records directly and support only explicitly pinned Qiskit versions.
- [x] Reuse existing target conversion behavior where correct, including named nodes, supported loci, native operations, timing, and calibration; keep any Qiskit index representation private to the helper.
- [x] Return an honest partial target when optional qhw fields are absent and fail rather than inventing all-to-all connectivity or calibration.
- [x] Keep service resolution, credentials, reservations, route policy, compilation, and submission out of the helper.
- [x] Add representative qhw fixtures and isolated tests for named nodes, directed and undirected edges, locus-specific operations, optional calibration, and missing required data.
- [x] Document that the helper is transitional and removable when standardized qhw is consumed directly by Qiskit integration.

Commit boundary: reusable qhw-to-Qiskit conversion in QFw.

### A02-PH4-C2. QFw qhw-to-MQT Target Helper

Design sections: [MQT Core Compiler Integration](design-amendment-02-compiler-integration.md#mqt-core-compiler-integration) and [Required QFw Work](design-amendment-02-compiler-integration.md#required-qfw-work).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/QFw`.

Checklist:

- [x] Create a public backend-independent qhw-to-MQT helper parallel to the Qiskit helper.
- [x] Accept the same qhw device, coupling, and calibration records directly and support only explicitly pinned MQT Core versions.
- [x] Construct MQT topology, native operation, supported-locus, parameter, and optional calibration data through public MQT APIs.
- [x] Keep MQT-specific node/index representations private to the helper and never return a replacement canonical qhw model.
- [x] Fail clearly when required qhw information cannot construct the qualified MQT target; do not substitute a generic architecture.
- [x] Keep service resolution, credentials, reservations, route policy, compilation, and submission out of the helper.
- [x] Add the same representative qhw fixture cases used by the Qiskit helper and document the transitional removal condition.

Commit boundary: reusable qhw-to-MQT conversion in QFw.

### A02-PH4-C3. QFw Target Helpers Phase Gate

Design section: [Required QFw Work](design-amendment-02-compiler-integration.md#required-qfw-work).

Checklist:

- [x] Add one QFw phase test that feeds identical qhw records to both helpers without a runtime, reservation, credential, backend, compiler, or device connection.
- [x] Assert neither helper imports or calls QRI execution or service-resolution modules.
- [x] Assert named-node and structured-operation qhw data is not mutated.
- [x] Assert exact pinned dependency versions and clear unsupported-version failures.
- [x] Run the legacy `QFwBackend` target and regression suites to prove extraction did not change legacy behavior.
- [x] Create the QFw companion phase-test procedure with exact commands for both optional dependency environments.

Commit boundary: Phase 16 isolated target-helper proof and legacy regression evidence.

## Phase 5

### A02-PH5-C1. Qiskit Compiler Package and Registration

Design sections: [Qiskit Compiler Integration](design-amendment-02-compiler-integration.md#qiskit-compiler-integration) and [Discrete Version Registration](design-amendment-02-compiler-integration.md#discrete-version-registration).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/libquantum-compiler-qiskit`.

Checklist:

- [x] Create the standalone repository, `pyproject.toml`, `src/libquantum_compiler_qiskit/`, package tests, README, license, and CI without modifying libquantum core packaging.
- [x] Package `compiler.json` beside `plugin.py`; use compiler ID `qiskit`, the libquantum plugin API version, and a factory with no import-time registration side effects.
- [x] Implement `plugin_path()` using installed package resources so callers can add the manifest directory to `InitConfig.compiler_paths`.
- [x] Keep Qiskit and QFw imports inside this optional package and declare exact dependency pins for each qualified release.
- [x] Register `QuantumCircuit`, its exact installed Qiskit versions, the Qiskit object representation, exact compiler input/output representations, and serializers.
- [x] Implement import analysis in `analysis.py` to extract logical qubits and required features without mutating the circuit or using qhw data.
- [x] Add package-isolation tests proving libquantum imports without this distribution and this distribution reports unavailable for every unlisted Qiskit version.

Commit boundary: installable Qiskit compiler package, manifest, registration, and import analysis.

### A02-PH5-C2. Qiskit Compilation Wrapper

Design sections: [Qiskit Target Construction](design-amendment-02-compiler-integration.md#target-construction), [Compilation](design-amendment-02-compiler-integration.md#compilation), and [Artifact Submission](design-amendment-02-compiler-integration.md#artifact-submission).

Checklist:

- [x] Freeze the exact Qiskit version, QFw helper version, input representation, output representation/version, serializer, and QPM acceptance contract in a package test matrix before implementing compilation; do not guess a range or fallback format.
- [x] In `compiler.py`, pass the request's qhw records directly to the public QFw qhw-to-Qiskit helper and build or retrieve the target using the full exact-version/device-generation cache key.
- [x] Transpile a preserved copy of the source circuit with deterministic qualified settings and no QRI call.
- [x] In `serialization.py`, emit only the negotiated exact exchange format and preserve layout, parameter identity, measurement interpretation, and required metadata.
- [x] Return authoritative `CompilerResult` payload, representation, snapshots, layout, parameter metadata, provenance, and safe diagnostics.
- [x] Report target-helper, unsupported-operation/locus, transpilation, and serialization failures at the compilation boundary without selecting another route.
- [x] Add tests proving original qhw object identity reaches the helper and the source circuit is unchanged.

Commit boundary: target-aware Qiskit compilation and exact artifact serialization.

### A02-PH5-C3. Qiskit Compiler Phase Gate

Design section: [Qiskit Tests](design-amendment-02-compiler-integration.md#qiskit-tests).

Checklist:

- [x] Add an isolated end-to-end package test using representative qhw records and a dummy QPM input contract; require no QFw service, reservation, credential, network, or hardware.
- [x] Prove exact-version registration, import analysis, target-helper invocation, topology/locus enforcement, deterministic compilation, metadata preservation, exact serialization, and zero QRI calls.
- [x] Install the built wheel into an isolated target and load its packaged manifest through libquantum's normal filesystem loader using `plugin_path()`.
- [x] Run the package unit suite and the libquantum Phase 15 gate with only this compiler enabled.
- [x] Create the repository's phase-test document with exact install and pytest commands and expected output.

Commit boundary: Phase 17 independently installable Qiskit compiler proof.

## Phase 6

### A02-PH6-C1. MQT Compiler Package and Registration

Design sections: [MQT Core Compiler Integration](design-amendment-02-compiler-integration.md#mqt-core-compiler-integration) and [Discrete Version Registration](design-amendment-02-compiler-integration.md#discrete-version-registration).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/libquantum-compiler-mqt`.

Checklist:

- [x] Create the standalone repository, `pyproject.toml`, `src/libquantum_compiler_mqt/`, package tests, README, license, and CI without modifying libquantum core packaging.
- [x] Package `compiler.json` beside `plugin.py`; use compiler ID `mqt`, the libquantum plugin API version, and a factory with no import-time registration side effects.
- [x] Implement `plugin_path()` using installed package resources so callers can add the manifest directory to `InitConfig.compiler_paths`.
- [x] Keep MQT Core, Qiskit conversion support, and QFw imports inside this optional package and declare exact dependency pins for each qualified release.
- [x] Register each supported MQT-native object type separately; register `QuantumCircuit` only for explicitly qualified Qiskit/MQT version combinations and mark conversion mode accurately.
- [x] Consume requirements stored by the active exact-version Qiskit object registration when present; in the separate qualified MQT environment provide an equivalent non-mutating import analyzer, and never analyze again during route listing or compilation.
- [x] Add package-isolation tests proving libquantum imports without this distribution and this distribution reports unavailable for every unlisted version combination.

Commit boundary: installable MQT compiler package, manifest, registration, and import analysis.

### A02-PH6-C2. MQT Compilation Wrapper

Design sections: [MQT Core Compiler Integration](design-amendment-02-compiler-integration.md#mqt-core-compiler-integration) and [Artifact Submission](design-amendment-02-compiler-integration.md#artifact-submission).

Checklist:

- [x] Freeze the exact MQT Core version, any Qiskit input version, QFw helper version, input representations, output representation/version, serializer, and QPM acceptance contract in a package test matrix before coding compilation.
- [x] In `compiler.py`, pass the request's qhw records directly to the public QFw qhw-to-MQT helper and build or retrieve the target with an exact-version/device-generation cache key.
- [x] Perform Qiskit-to-MQT conversion only on a declared converted-input path and record that stage in provenance.
- [x] Invoke the qualified MQT compilation API without QRI calls and preserve source semantics, placement/layout, parameters, and selected objective/configuration.
- [x] In `serialization.py`, emit only the negotiated exact exchange representation/version accepted by the QPM.
- [x] Return an authoritative `CompilerResult` and report target-helper, conversion, compilation, and serialization failures without route fallback.
- [x] Add tests proving original qhw object identity reaches the helper and source inputs remain unchanged.

Commit boundary: target-aware MQT compilation and exact artifact serialization.

### A02-PH6-C3. MQT Compiler Phase Gate

Design section: [MQT Tests](design-amendment-02-compiler-integration.md#mqt-tests).

Checklist:

- [x] Add isolated native-MQT and qualified Qiskit-origin end-to-end tests using representative qhw records and a dummy QPM input contract.
- [x] Prove exact version-pair registration, import requirements reuse, qhw helper invocation, topology/native-operation enforcement, conversion provenance, metadata preservation, exact serialization, and zero QRI calls.
- [x] Install the built wheel into an isolated target and load its packaged manifest through libquantum's normal loader using `plugin_path()`.
- [x] Run the package unit suite and the libquantum Phase 15 gate with only this compiler enabled.
- [x] Create the repository's phase-test document with exact install and pytest commands and expected output.

Commit boundary: Phase 18 independently installable MQT compiler proof.

## Phase 7

### A02-PH7-C1. Compiler Packages Isolated Integration

Design sections: [Test Strategy](design-amendment-02-compiler-integration.md#test-strategy) and [Acceptance Criteria](design-amendment-02-compiler-integration.md#acceptance-criteria).

The qualified Qiskit 2.2.3 and MQT/Qiskit 2.5.2 dependency sets intentionally use separate clean environments, matching QFw-SLURM-Cluster.

Checklist:

- [x] Build wheels for libquantum, QFw target helpers, `libquantum-compiler-qiskit`, and `libquantum-compiler-mqt`; install the Qiskit and MQT compiler packages into their separate exact-dependency clean environments.
- [x] Prove libquantum alone imports without either compiler package.
- [x] Load each packaged manifest independently through `plugin_path()` and the normal libquantum loader.
- [x] Run direct Qiskit and MQT compilation routes over identical representative qhw records and dummy QPM input advertisements.
- [x] Prove import analysis occurs once, route listing performs no compiler callback, and compilation performs no QRI submission.
- [x] Prove exact compiler results pass unchanged into QRI submission and qhw objects reach target helpers unchanged.
- [x] Retain exact dependency versions, manifests, route map, qhw fixtures, compiler traces, outputs, and cleanup trace.

Commit boundary: isolated composition proof for both standalone compiler packages.

### A02-PH7-C2. Compiler Deployment Qualification

Design section: [QFw Integration Qualification](design-amendment-02-compiler-integration.md#qfw-integration-qualification).

Status: simulator deployment qualification is complete in QFw-SLURM-Cluster. Physical-device qualification remains separate and is not claimed.

Checklist:

- [x] Freeze exact QFw, libquantum, compiler-package, Qiskit, MQT Core, qhw schema, QPM provider, and accepted-input revisions.
- [x] Replace dummy query/qhw telemetry and target helpers with the deployed paths one boundary at a time.
- [x] Qualify at least one exact Qiskit artifact and one exact MQT artifact accepted through QRI by the selected QPM.
- [x] Record layouts, parameter metadata, compiler provenance, QPM-side additional stages, operation IDs, and returned qhw results.
- [x] Keep Catalyst unavailable and do not count Backline direct execution as a compiler route.
- [x] Distinguish simulator evidence from physical-device evidence and do not count skipped gates as qualification.
- [x] Create the deployment procedure and retain logs without credentials or provider secrets.

Evidence: [Amendment 01 Phase 4 QFw-SLURM-Cluster Qualification](phase-tests/amendment-01-phase-04-qfw-slurm-cluster.md) freezes the exact package and source revisions. NWQSim Slurm job 41 qualifies the exact Qiskit compiler path through `QFwBackendV2`; job 37 qualifies a native MQT Core program through the exact MQT compiler, QRI, and the selected NWQSim QPM. Retained logs include manifests, routes, layouts, parameter metadata, provenance, operation IDs, and results.

Commit boundary: frozen deployed compiler-integration qualification evidence.

### A02-PH7-C3. Compiler Integration Traceability Audit

Design section: the complete compiler integration amendment.

Checklist:

- [x] Map every amendment statement to one implementation unit and executable test.
- [x] Confirm no source, test, or public export retains the separate input-capability RPC, integer qhw coupling reduction, operation-name-only target model, route-time compatibility callback, or compiler-result validation.
- [x] Confirm libquantum core contains no Qiskit, MQT, QFw backend, or compiler-package imports.
- [x] Confirm both compiler packages are separate repositories with exact version matrices and no undeclared dependency on each other.
- [x] Confirm QFw target helpers are transitional and helper-specific types do not enter libquantum records.
- [x] Confirm every completed checkbox has evidence and every deployment-only item remains unchecked with a reason.
- [x] Render this plan and record the final commands and results.

Commit boundary: amendment-specific compiler integration and anti-drift audit.
