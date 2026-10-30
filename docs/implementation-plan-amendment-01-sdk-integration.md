# SDK Integration Amendment Implementation Plan

**Status:** draft

## A01-GUIDANCE. Instructions for the Implementation Agent

Before coding, read the complete amendment, the current unit, and every linked dependency. Keep this plan open throughout implementation. Work through units and checklist items in dependency order, even when a dependency is defined in the Compiler Integration Amendment Implementation Plan. Complete the dependency's implementation units and gates first, update and render that plan, then return to the blocked SDK-integration unit. Within units at the same dependency level, preserve the documented unit and checklist order. Check a box only after its implementation and stated evidence are complete.

At the end of every unit, update the owning JSONL source, render its Markdown companion with the ElectroBoy artifact renderer, and include the checklist update in the same commit pass. One unit is one reviewable commit pass. Do not start a dependent unit while its prerequisite has unchecked implementation work.

Do not silently skip, combine, reinterpret, or expand tasks. If code, another plan, or an upstream contract conflicts with the amendment, stop at the affected checkbox and update the governing design or plan before continuing.

## A01-VERIFICATION. Phase Verification Contract

Every phase ends with a runnable gate and companion test procedure in the owning repository. The procedure records linked design sections, prerequisites, fixtures, exact commands, assertions, expected output, cleanup, and retained evidence.

Isolated gates require no network, credentials, allocation, or hardware. Deployment gates are explicitly marked and a skip is not qualification evidence. Fix every defect discovered by a gate before checking the phase complete.

## A01-SCOPE. Scope, Ownership, and Dependencies

This plan implements the active QPM discovery and Qiskit portions of [Design Amendment 01: SDK Integration and QPM-Driven Discovery](design-amendment-01-sdk-integration.md).

Repository ownership:

- `/home/a2e/ORNL/Quantum/openQSE/QFw` owns `QPM.query()`, QPM provider population, `QFwBackendV2`, `QFwJobV2`, Qiskit result conversion, and legacy-backend coexistence.
- `/home/a2e/ORNL/Quantum/openQSE/libquantum` consumes the authoritative query response, retains allocation bindings, builds route maps, and tracks operations.
- [Compiler Integration Amendment Implementation Plan](implementation-plan-amendment-02-compiler-integration.md) owns unchanged qhw retention, target helpers, and independently packaged Qiskit/MQT compilers.

Dependency precedence overrides document number. Before starting any SDK-integration checklist item, complete every compiler-plan unit and phase gate that the item depends on. Return to this plan only after those prerequisite checkboxes are complete. Do not duplicate compiler implementation in this plan. The application sees ordinary Qiskit APIs and never supplies a libquantum route, QPM, provider, hardware type, queue, or operation identifier.

## A01-FUTURE. Future Scope

PennyLane support and `QFwDevice` are postponed to future work. The PennyLane sections of Design Amendment 01 are not implemented by this plan, are not prerequisites for Qiskit completion, and provide no current checklist activities or qualification claims.

A future design revision must define the PennyLane compiler-input path, supported versions and operations, result contract, and implementation ownership before a separate implementation plan is created.

## Phase 1

### A01-PH1-C1. QPM Query Contract

Design sections: [Authoritative QPM Query Contract](design-amendment-01-sdk-integration.md#authoritative-qpm-query-contract), [Accepted Program Inputs](design-amendment-01-sdk-integration.md#accepted-program-inputs), and [Required QFw Changes](design-amendment-01-sdk-integration.md#required-qfw-changes).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/QFw`.

Checklist:

- [x] Add `program_inputs` to the existing `DEFwServiceInfo.properties` returned by each supported QPM `query()`; do not introduce a replacement response type or a second capability RPC.
- [x] Populate service ID, provider, device ID, QPM type/capability flags, hardware/simulator identity, physical capacity, and exact program-input entries from the actual execution path.
- [x] Represent every accepted format with exact representation, exact versions, preference, and native flag; advertise nothing that submission cannot accept.
- [x] Keep telemetry method availability and credentials out of the query properties.
- [x] Update IQM, IBM, simulator, and dummy QPM implementations that participate in qualification, recording unsupported providers explicitly rather than fabricating defaults.
- [x] Add QFw contract fixtures for single and multiple reservation/service pairs and assert that `query()` returns the exact reserved service identity.
- [x] Add one submission contract test per advertised representation/version and reject an advertisement whose execution path is absent.
- [x] Preserve existing `DEFwServiceInfo` callers and the legacy backend regression suite.

Commit boundary: QFw query contract and provider population, independent of libquantum changes.

### A01-PH1-C2. Libquantum Query Consumption

Design sections: [Allocation-Scoped QPM Discovery](design-amendment-01-sdk-integration.md#allocation-scoped-qpm-discovery), [Authoritative QPM Query Contract](design-amendment-01-sdk-integration.md#authoritative-qpm-query-contract), and [libquantum Initialization](design-amendment-01-sdk-integration.md#libquantum-initialization).

Current implementation affected: `libquantum/_qfw_runtime.py`, `libquantum/_qri.py`, `tests/fixtures/dummy_qri.py`, and discovery tests.

Checklist:

- [x] Preserve exact reservation/service resolution from the active QFw allocation; require no application device selector.
- [x] Replace `get_program_input_capabilities()` with one `QPM.query()` call and read exact `program_inputs` from the existing `DEFwServiceInfo.properties`.
- [x] Extract service identity, provider, device identity, QPM type/capabilities, hardware/simulator identity, physical capacity, and accepted program inputs required for route construction.
- [x] Continue obtaining usable capacity and qhw device/coupling/calibration records through the existing telemetry calls owned by the compiler-integration plan.
- [x] Construct routes only from exact compiler-output and QPM-input representation/version matches.
- [x] Update the dummy QPM to return `DEFwServiceInfo`-shaped query properties and remove the separate capability RPC.
- [x] Preserve opened route bindings across refresh and translate only failures that prevent discovery or mapping.

Commit boundary: libquantum consumption of the authoritative QPM query contract.

### A01-PH1-C3. Query Discovery Phase Gate

Design sections: [Allocation-Scoped QPM Discovery](design-amendment-01-sdk-integration.md#allocation-scoped-qpm-discovery) and [Validation](design-amendment-01-sdk-integration.md#validation).

Checklist:

- [x] Create `tests/phase_tests/test_amendment_01_phase_01_query_discovery.py`.
- [x] Prove one and multiple reserved services are discovered without provider/device arguments.
- [x] Prove exact query-advertised inputs control route construction and the removed RPC is never called.
- [x] Prove service identity conflicts and missing required mapping fields produce service-specific discovery failures.
- [x] Prove refresh retains already opened route bindings.
- [x] Create `docs/phase-tests/amendment-01-phase-01-query-discovery.md` and run the documented pytest command.

Commit boundary: Phase 1 executable query-driven discovery proof.

## Phase 2

### A01-PH2-C1. QFwBackendV2 Initialization and Target

Design sections: [New Qiskit Backend Integration](design-amendment-01-sdk-integration.md#new-qiskit-backend-integration), [Backend Ownership and Initialization](design-amendment-01-sdk-integration.md#backend-ownership-and-initialization), [Qiskit Target Construction](design-amendment-01-sdk-integration.md#qiskit-target-construction), and [Coexistence and Migration](design-amendment-01-sdk-integration.md#coexistence-and-migration).

Owning repository: `/home/a2e/ORNL/Quantum/openQSE/QFw`.

Checklist:

- [x] Add new `QFwBackendV2` and `QFwJobV2` modules, exports, provider identifiers, and tests without modifying `QFwBackend` or `QFwJob` behavior.
- [x] Make the public backend constructor accept only Qiskit-facing policy/timeouts; expose no libquantum context, route, QPM, provider, hardware type, or device selector.
- [x] Resolve installed compiler package paths internally from deployment configuration or `libquantum_compiler_qiskit.plugin_path()` and initialize libquantum from the active allocation.
- [x] Query structural Qiskit routes, apply a deterministic backend-owned compiler policy, report ambiguity, and open exactly one route; libquantum must not choose it.
- [x] Build the backend `Target` from the route's retained qhw records through the public QFw qhw-to-Qiskit helper.
- [x] Finalize a partially initialized context on every constructor failure.
- [x] Keep dependency direction `QFw adapter -> libquantum -> QRI/QPM` and add a source check preventing libquantum imports of QFw SDK adapters.

Commit boundary: coexisting Qiskit V2 backend public surface, initialization, route, and target.

### A01-PH2-C2. QFwJobV2 Operation Lifecycle

Design sections: [Backend Execution](design-amendment-01-sdk-integration.md#backend-execution), [Qiskit Job Ownership](design-amendment-01-sdk-integration.md#qiskit-job-ownership), and [Status and Cancellation](design-amendment-01-sdk-integration.md#status-and-cancellation).

Checklist:

- [x] Normalize one circuit or a circuit batch and reject unsupported run options before remote work.
- [x] Import and compile the entire batch before creating remote operations so a later local compilation failure leaves no partial remote batch.
- [x] Give each `QFwJobV2` one completion queue and an ordered per-circuit record containing its private libquantum `op_id`, completion, and submission error.
- [x] Submit compiled results in order; retain every accepted `op_id` if a later immediate submission fails and never cancel implicitly.
- [x] Aggregate `Context.status(op_id)` into Qiskit `JobStatus` using the amendment table while preserving unknown QPM states for diagnostics.
- [x] Implement `cancel()` across unfinished operations and report cancelled only after terminal cancellation confirmation.
- [x] Keep libquantum routes, queues, and operation identifiers out of ordinary application APIs and results.

Commit boundary: QFwJobV2 compilation, submission, status, and cancellation.

### A01-PH2-C3. QFwBackendV2 Results and Shutdown

Design sections: [Result Collection](design-amendment-01-sdk-integration.md#result-collection) and [Backend Shutdown](design-amendment-01-sdk-integration.md#backend-shutdown).

Checklist:

- [x] Correlate out-of-order completions by `op_id` and restore original circuit order.
- [x] Convert only qualified qhw/QRI result forms into Qiskit counts, memory, statevector, metadata, and `Result`; preserve bit order, shots, and experiment identity.
- [x] Keep timed-out jobs alive with their queue and operation records so later result, status, or cancellation remains possible.
- [x] Cache the terminal Qiskit result, close the queue after all completions are consumed, and make repeated `result()` calls local.
- [x] Implement idempotent backend shutdown through `libquantum.finalize(timeout)`; report outstanding operation IDs and never cancel implicitly.
- [x] Run the unchanged legacy backend suite in the same installation and prove no fallback between V2 and legacy backends.

Commit boundary: Qiskit result conversion, timeout retention, shutdown, and coexistence.

### A01-PH2-C4. QFwBackendV2 Phase Gate

Design sections: [New Qiskit Backend Integration](design-amendment-01-sdk-integration.md#new-qiskit-backend-integration) and [Validation](design-amendment-01-sdk-integration.md#validation).

Checklist:

- [x] Add isolated `QFwBackendV2` tests using libquantum dummy QPM/events and the installed Qiskit compiler package; require no network, reservation service, credential, or hardware.
- [x] Exercise direct backend `run()`, `BackendEstimatorV2`, and `BackendSamplerV2` for the qualified Qiskit version.
- [x] Exercise batch compilation-before-submission, partial immediate submission failure, out-of-order completions, status aggregation, confirmed cancellation, result timeout/retry, and shutdown timeout.
- [x] Assert application code sees only Qiskit objects and never supplies device type, provider, route, queue, or operation ID.
- [x] Run both V2 and legacy backend suites in the same environment.
- [x] Create the QFw phase-test procedure with exact commands and expected results.

Commit boundary: Phase 19 runnable Qiskit SDK integration proof.

## Phase 3

### A01-PH3-C1. Qiskit Adapter Isolated Integration

Design sections: [New Qiskit Backend Integration](design-amendment-01-sdk-integration.md#new-qiskit-backend-integration) and [Coexistence and Migration](design-amendment-01-sdk-integration.md#coexistence-and-migration).

Dependencies: completed isolated Qiskit gates from [Compiler Integration Amendment Implementation Plan](implementation-plan-amendment-02-compiler-integration.md).

Checklist:

- [x] Build wheels for libquantum, QFw, and the Qiskit compiler package and install them into a clean environment.
- [x] Run `QFwBackendV2` through direct backend execution, `BackendEstimatorV2`, and `BackendSamplerV2` over dummy QRI.
- [x] Assert the Qiskit application sees no libquantum object, route, QPM, provider selector, queue, or operation ID.
- [x] Assert the adapter restores user-visible circuit and result order after out-of-order completions.
- [x] Run the unchanged legacy `QFwBackend` regression suite in the same installation.
- [x] Retain package versions, query properties, route choice, operation trace, completion trace, and cleanup trace.

Commit boundary: isolated multi-package Qiskit integration evidence.

### A01-PH3-C2. Qiskit Adapter Deployment Qualification

Design section: [Validation](design-amendment-01-sdk-integration.md#validation).

Checklist:

- [x] Freeze exact QFw, libquantum, Qiskit-adapter, Qiskit-compiler, QPM, and qhw revisions.
- [x] Replace dummy boundaries one at a time with allocation resolution, real query/telemetry, the Qiskit compiler package, QRI execution, completions, status, and cancellation.
- [x] Qualify `QFwBackendV2()` without provider/device arguments on at least one advertised QPM route.
- [x] Prove finalization timeout reports exact outstanding operation IDs without implicit cancellation.
- [x] Distinguish simulator evidence from physical-device evidence and do not count a skipped test as qualification.
- [x] Create the deployment procedure and retain logs without credentials or remote service objects.

Evidence: [Amendment 01 Phase 4 QFw-SLURM-Cluster Qualification](phase-tests/amendment-01-phase-04-qfw-slurm-cluster.md) freezes the deployed revisions and records NWQSim Slurm job 41. The live job proves allocation-scoped discovery, query/telemetry, compilation, QRI execution, status, completion, and cleanup. Exact isolated QFw gates cover cancellation and finalization-timeout semantics against the same adapter revision. This is simulator evidence only; no physical-device qualification is claimed.

Commit boundary: frozen deployed Qiskit-adapter qualification evidence.

### A01-PH3-C3. Qiskit Integration Traceability Audit

Design section: the active QPM discovery and Qiskit portions of the SDK integration amendment.

Checklist:

- [x] Map every active amendment statement to an implementation unit and executable test.
- [x] Confirm Qiskit applications never provide device type, provider, QPM, route, queue, or operation ID.
- [x] Confirm `QFwBackendV2` and `QFwJobV2` coexist with unchanged legacy classes and have no automatic fallback.
- [x] Confirm QFw adapters depend on libquantum and libquantum never imports SDK adapters.
- [x] Confirm every completed checkbox has evidence and every unresolved deployment item remains unchecked with a reason.
- [x] Render this plan and record the final commands and results.

Evidence: the isolated phase gates, the 98-test QFw legacy/V2 regression gate, and the retained live Slurm procedure cover the active Qiskit amendment. Physical-device qualification is outside the completed simulator scope and remains explicitly unclaimed.

Commit boundary: amendment-specific Qiskit integration and anti-drift audit.

## Phase 4

### A01-PH4-C1. QFw-SLURM-Cluster Dual-Backend System Qualification

Design sections: [Validation](design-amendment-01-sdk-integration.md#validation) and [Coexistence and Migration](design-amendment-01-sdk-integration.md#coexistence-and-migration).

Owning repositories: `/home/a2e/ORNL/Quantum/openQSE/QFw`, `/home/a2e/ORNL/Quantum/openQSE/QFw-release-v0.1/repos-main-sync/qfw-slurm`, and `/home/a2e/ORNL/Quantum/openQSE/QFw-release-v0.1/repos-main-sync/QFw-SLURM-Cluster`.

Dependencies: complete the compiler-package isolated integration gate, the `QFwBackendV2` phase gate, and the Qiskit adapter isolated integration gate before changing the deployment.

Checklist:

- [x] Freeze and record the exact QFw, qfw-slurm, QFw-SLURM-Cluster, libquantum, Qiskit compiler, Qiskit, QPM, and qhw revisions used by the system test.
- [x] Extend the QFw-SLURM-Cluster build and provisioning path to build wheels for libquantum and the Qiskit compiler package, install them into the activated QFw virtual environment, and verify their packaged manifests and imports from an ordinary Slurm application process.
- [x] Preserve qfw-slurm's canonical `QFW_RESERVATIONS` injection and QFw activation contracts; prove `QFwBackendV2` resolves only the services reserved for the running allocation.
- [x] Add one shared command-line backend selector for QFw Qiskit examples under `QFw/examples/tests`, with explicit `--backend-api legacy` and `--backend-api v2` values and no automatic fallback.
- [x] Make every Qiskit example that constructs a QFw backend use the shared selector; retain legacy provider arguments only on the legacy path, and construct the V2 path without provider, device, route, queue, or operation identifiers.
- [x] Run the same deterministic circuit and shots through both backend modes; because NWQSim exposes no seed control, compare supported operations, transpilation behavior, batch ordering, GHZ count invariants, metadata, status, cancellation, timeout handling, and cleanup across the live and exact isolated gates.
- [x] Exercise both modes through the QFw-SLURM-Cluster Slurm launch path against at least one simulator QPM; label any physical-device run separately and do not treat a skip as qualification.
- [x] Assert the selected backend class and exact package versions are present in each retained log, and fail if either mode silently falls back to the other.
- [x] Run the unchanged qfw-slurm gateway/protocol regression gates and the QFw legacy/V2 isolated gates after cluster integration changes.
- [x] Create a companion system-test procedure with exact cluster build, allocation, launch, comparison, cleanup, and log-sanitization commands.

Evidence: [Amendment 01 Phase 4 QFw-SLURM-Cluster Qualification](phase-tests/amendment-01-phase-04-qfw-slurm-cluster.md) records build/install verification, qfw-slurm's 7-test regression gate, QFw's 98-test legacy/V2 gate, and completed NWQSim Slurm job 41. The route is simulator-only.

Commit boundary: reproducible QFw-SLURM-Cluster dual-backend system qualification.
