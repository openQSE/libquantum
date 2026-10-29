# Implementation Plan

**Status:** draft

## PLAN-GUIDANCE. Instructions for the Implementation Agent

Before coding, read the current phase, every linked detailed-design section, and the listed requirements. Keep this plan open while implementing. Work through units and checklist items in order. Check a box only after the implementation and its stated evidence are complete. At the end of every unit, update this source artifact to mark completed tasks, render it with `electroboy render-artifact implementation-plan`, and include that update in the unit's commit pass.

Do not silently skip, reinterpret, combine, or expand a task. If source evidence conflicts with the plan or design, stop at the affected task, record the discrepancy, and update the governing requirements, design, or plan artifact before continuing. Report blocked and deferred items with a concrete reason. One unit is one reviewable commit pass; do not begin the next unit until the current unit's checklist and phase evidence are internally consistent.

## PLAN-VERIFICATION. Phase Verification Contract

Each phase ends with a runnable phase gate. The implementation creates one executable module at `tests/phase_tests/test_phase_<NN>_<name>.py` and one companion procedure at `docs/phase-tests/phase-<NN>-<name>.md`. The document must state the linked design sections and requirements, prerequisites, fixture inputs, exact command, assertions and expected output, cleanup, failure interpretation, and evidence retained for review. The standard command is `python -m pytest -q <phase-test-module>`.

A phase gate must run directly and deterministically after its phase is implemented. It may depend on artifacts completed by earlier phases, but it must not require later phases. Phases through isolated end-to-end validation use repository-owned dummy integrations and require no compiler SDK, QFw service, credentials, network, reservation, or hardware. Real QFw qualification is a separate deployment gate. Every defect found by a phase gate is fixed within that phase before its checklist is checked complete.

## PLAN-SCOPE. Scope and Traceability

This plan implements the Python prototype defined by the detailed design. Route eligibility belongs to libquantum; route choice belongs to the application. Compilation and submission are separate. `run()` submits an existing compiler result or explicit opaque program, returns QRI `cid` as `op_id`, and tracks that operation internally for QPM-backed status, cancellation, completion, and finalization.

Automatic route-selection policies, compiler configuration/pass pipelines, graph/executor layers, result transformation, runtime plugin installation, and broad SC26 qualification remain future work until their design sections are revised. At final coverage review, classify every requirement as implemented by a linked design section and phase gate or explicitly deferred by [Future Requirements](detailed-design.md#future-requirements). Any unclassified requirement is a blocking artifact discrepancy.

## Phase 1

### PH1-C1. Public API Surface

Design sections: [Application API](detailed-design.md#application-api) and [Application Workflow](detailed-design.md#application-workflow).

Requirements: `LQ-INT-002`, `LQ-INT-003`, `LQ-APP-001`, `LQ-APP-002`.

Checklist:

- [ ] Create the Python package boundary and export only the public names declared by the design.
- [ ] Declare `initialize(config=None)` and `finalize(context, timeout=None)` with their designed contracts.
- [ ] Declare every `Context` method with the exact designed parameters and return type, including separate `compile()`, submission-only `run()`, `status(op_id)`, and `cancel(op_id)`.
- [ ] Declare stateful `Context`, `ProgramRoute`, and `CompletionQueue` handles without exposing constructors for context-owned handles.
- [ ] Declare immutable argument and result records referenced directly by the API, using forward references until their owning phase implements them.
- [ ] Keep graph, executor, result-processing, circuit-cutting, runtime-plugin-installation, automatic selection, compiler configuration, and future compatibility APIs out of the exported SC26 surface.
- [ ] Make unimplemented method bodies fail explicitly as internal development placeholders; do not return plausible dummy success from the public library.

Commit boundary: the importable public surface and exact signatures only.

### PH1-C2. Application API Phase Gate

Design sections: [Application API](detailed-design.md#application-api) and [Application Workflow](detailed-design.md#application-workflow).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_01_application_api.py`.
- [ ] Verify the package imports without optional SDK, compiler, QFw, or device dependencies installed.
- [ ] Verify every designed public symbol and method signature, including `run()` returning `str` and status/cancellation accepting `op_id`.
- [ ] Verify context-owned handle constructors are not public application APIs.
- [ ] Verify excluded future APIs are absent from the exported surface.
- [ ] Create `docs/phase-tests/phase-01-application-api.md` using the phase verification contract.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_01_application_api.py` and record the passing evidence.

Commit boundary: Phase 1 executable contract and its test document.

## Phase 2

### PH2-C1. Common and Program Records

Design sections: [Shared Data Model](detailed-design.md#shared-data-model) and [Program Representation](detailed-design.md#program-representation).

Requirements: `LQ-PRG-001` through `LQ-PRG-005`, `LQ-PRG-007`, `LQ-CON-006`.

Checklist:

- [ ] Implement immutable `VersionSet` and `Diagnostic` records.
- [ ] Implement `ProgramKind`, `Representation`, and their field validation.
- [ ] Implement immutable `ProgramRequirements` with caller constraints preserved.
- [ ] Implement immutable `Program` while retaining the exact original object or payload identity.
- [ ] Enforce lowercase namespaced symbolic identifiers while preserving case in derived Python type identities.
- [ ] Use only exact reported versions; add no range, adjacency, or compatibility inference.
- [ ] Keep parameter bindings, shots, route destination, and compiler configuration out of reusable program state.

Commit boundary: common and program value records.

### PH2-C2. Device, Route, and Operation Records

Design sections: [Device Representation](detailed-design.md#device-representation), [Program Route](detailed-design.md#program-route), and [Operation Records](detailed-design.md#operation-records).

Requirements: `LQ-DEV-001` through `LQ-DEV-004`, `LQ-FMT-005`, `LQ-OBS-002`.

Checklist:

- [ ] Implement `FormatSupport`, `DeviceCapacity`, coupling records, and immutable public `Device`.
- [ ] Implement private `DeviceAccessContext`; keep reservation, QPM service, binding objects, and credentials out of `Device`.
- [ ] Implement immutable `AvailableRoute` and a context-owned `ProgramRoute` that references its shared definition.
- [ ] Expose designed route properties while keeping compiler state, callbacks, and device access private.
- [ ] Implement `RouteConstraints`, `ExecutionOptions`, and `CompilerProvenance` with exact designed defaults.
- [ ] Ensure multiple routes can reference one immutable device snapshot.
- [ ] Ensure callers cannot forge or mutate context-owned handles.

Commit boundary: device, route, and operation records.

### PH2-C3. Data Model Phase Gate

Design sections: [Shared Data Model](detailed-design.md#shared-data-model), [Program Representation](detailed-design.md#program-representation), [Device Representation](detailed-design.md#device-representation), [Program Route](detailed-design.md#program-route), and [Operation Records](detailed-design.md#operation-records).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_02_data_model.py`.
- [ ] Exercise immutability, defaults, exact-version behavior, payload identity, private access separation, shared device references, and distinct route instance identity.
- [ ] Exercise invalid identifiers, invalid versions, forged handles, and mutation attempts as explicit failures.
- [ ] Create `docs/phase-tests/phase-02-data-model.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_02_data_model.py` and record the passing evidence.

Commit boundary: Phase 2 executable data-model contract and its test document.

## Phase 3

### PH3-C1. Compiler Callback Contract

Design section: [Compiler Integration Contract](detailed-design.md#compiler-integration-contract).

Requirements: `LQ-CMP-001` through `LQ-CMP-005`, `LQ-COMP-001`, `LQ-COMP-002`, `LQ-EXT-006`, `LQ-CON-007`.

Checklist:

- [ ] Implement compiler initialization, identity, object registration, capability, analysis, compatibility, request/result, callback, registration, manifest, and factory contracts.
- [ ] Validate exact representation/version sets and nonnegative preference ranks.
- [ ] Enforce native input/output preference ordering and the advertised input-to-output cross-product invariant.
- [ ] Pass only context identity and deployment-selected compiler configuration to compiler initialization.
- [ ] Keep compiler options and pass pipelines out of the SC26 callback request.
- [ ] Define callback failure translation points without swallowing integration errors.

Commit boundary: compiler-facing contract types and invariants.

### PH3-C2. Compiler Plugin Loader

Design section: [Compiler Integration Contract](detailed-design.md#compiler-integration-contract).

Checklist:

- [ ] Discover `compiler.json` only beneath configured compiler directories.
- [ ] Validate manifest schema/API version, ID, module, factory, and path containment before importing code.
- [ ] Load through the manifest-selected factory without module-import registration side effects.
- [ ] Reject duplicate compiler IDs, manifest/registration ID mismatch, missing factories, invalid callbacks, unsupported versions, path escapes, and unknown enabled IDs.
- [ ] Initialize enabled integrations and cache immutable compiler descriptions and exact capabilities for the current decision context.
- [ ] Validate contributed object registrations and serialized recognizers against the contributing compiler's inputs.
- [ ] Build the representation registry while retaining independent version support for the same exact Python type.
- [ ] Finalize initialized integrations in reverse order after success or partial failure.

Commit boundary: deterministic plugin discovery, lifecycle, and representation registry.

### PH3-C3. Dummy Compiler Integrations

Design section: [Compiler Integration Contract](detailed-design.md#compiler-integration-contract).

Checklist:

- [ ] Create a minimal dummy SDK object type used only by phase and isolated tests.
- [ ] Create dummy compiler A with native dummy-object input, one serialized recognizer, deterministic analysis/compatibility results, and one output representation.
- [ ] Create dummy compiler B with converted dummy-object input, a different exact version set, and a second compatible output route.
- [ ] Make both dummy compilers return their input wrapped in a deterministic marker payload; they must perform no real compilation.
- [ ] Record every callback invocation and received request so tests can assert dispatch and data propagation.
- [ ] Add configurable rejection, exception, unavailable, and cleanup-failure modes.
- [ ] Package each dummy compiler through the same manifest/factory path as a real installed integration.
- [ ] Keep dummy integrations out of production discovery unless their fixture directory is explicitly configured.

Commit boundary: reusable compiler contract fixtures, separate from production code.

### PH3-C4. Compiler Integration Phase Gate

Design section: [Compiler Integration Contract](detailed-design.md#compiler-integration-contract).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_03_compiler_integration.py`.
- [ ] Exercise manifest loading, exact capabilities, native/converted distinction, object and serialized recognition, ambiguity, unknown input, callback recording, deterministic no-op output, and reverse cleanup.
- [ ] Exercise every declared manifest, registration, lifecycle, and callback failure mode.
- [ ] Create `docs/phase-tests/phase-03-compiler-integration.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_03_compiler_integration.py` and record the passing evidence.

Commit boundary: Phase 3 executable compiler contract and its test document.

## Phase 4

### PH4-C1. Dummy QRI and Device Integrations

Design sections: [Device Discovery Through QRI](detailed-design.md#device-discovery-through-qri) and [Device Representation](detailed-design.md#device-representation).

Checklist:

- [ ] Create an in-memory dummy QFw runtime with allocation, reservation, service, execution, telemetry, and event-binding identities.
- [ ] Create at least two dummy devices in one allocation with different usable capacities and exact accepted input representation/version sets.
- [ ] Create one unrelated device in another allocation to prove allocation filtering.
- [ ] Return versioned QRI-shaped device capability data and mechanically translate it through the production adapter.
- [ ] Add deterministic snapshot generations and configurable metadata-incomplete, unavailable-service, malformed-capability, stale-snapshot, and access-mismatch modes.
- [ ] Create a dummy QPM execution endpoint that records submissions, returns deterministic `cid` values, implements `task_status(cid, reservation_id)`, implements `cancel_task(cid, reservation_id)`, and emits confirmed terminal cancellation events.
- [ ] Keep dummy runtime/device integrations behind explicit fixture construction; production discovery must never select them implicitly.

Commit boundary: reusable device, QRI, QPM, and event fixtures.

### PH4-C2. Allocation-Scoped Device Discovery

Design section: [Device Discovery Through QRI](detailed-design.md#device-discovery-through-qri).

Requirements: `LQ-INIT-004`, `LQ-RES-001`, `LQ-RES-002`, `LQ-DEV-001`, `LQ-DEV-002`, `LQ-COMP-003`, `LQ-QFW-001`, `LQ-QFW-003`, `LQ-QFW-005`.

Checklist:

- [ ] Resolve active runtime, allocation, reservation, QPM service, execution binding, and telemetry binding through the QFw adapter boundary.
- [ ] Query only devices authorized by the allocation.
- [ ] Translate QRI device descriptions and versioned input capabilities without changing field meaning.
- [ ] Preserve the private allocation/device/service/reservation association used by later route dispatch.
- [ ] Produce immutable `ResourceSnapshot` data with a deterministic generation.
- [ ] Implement refresh as a new snapshot; do not mutate existing devices or opened bindings.
- [ ] Reuse compatible lower-level QFw Backend helpers where contracts match and preserve that backend's independent behavior.
- [ ] Translate discovery failures with safe stage-specific context.

Commit boundary: production discovery adapter exercised through dummy QRI.

### PH4-C3. Device Discovery Phase Gate

Design section: [Device Discovery Through QRI](detailed-design.md#device-discovery-through-qri).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_04_device_discovery.py`.
- [ ] Prove allocation filtering, exact capability translation, private access binding, stable snapshots, refresh generation, incomplete metadata handling, and discovery error translation.
- [ ] Prove the unrelated dummy device cannot appear in the active allocation snapshot.
- [ ] Create `docs/phase-tests/phase-04-device-discovery.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_04_device_discovery.py` and record the passing evidence.

Commit boundary: Phase 4 executable discovery contract and its test document.

## Phase 5

### PH5-C1. Context Lifecycle

Design section: [Initialization and Finalization](detailed-design.md#initialization-and-finalization).

Requirements: `LQ-INIT-001` through `LQ-INIT-004`, `LQ-FIN-001` through `LQ-FIN-005`.

Checklist:

- [ ] Implement `InitConfig`, lifecycle states, `FinalizeResult.outstanding_op_ids`, and context identity.
- [ ] Initialize in the designed order: runtime/allocation access, plugin discovery, compiler state/capabilities, representation registry, resource snapshot, compatibility map, event support, then `READY`.
- [ ] Reject public operations unless the context state permits them and unwind failed initialization in reverse order.
- [ ] Stop new compile/submission calls atomically when finalization begins.
- [ ] Wait for internally tracked operations to become terminal until the supplied timeout, querying their owning QPM without consuming application completions.
- [ ] On timeout, emit a diagnostic listing every outstanding `op_id` and return `TIMED_OUT` with those IDs.
- [ ] Do not implicitly cancel outstanding operations or claim that they completed.
- [ ] Unregister endpoints, close library-owned queues, finalize compilers, and release library-owned connections after waiting.
- [ ] Never release launcher-owned allocation/reservation state or shut down shared QPM.
- [ ] Make repeated finalization safe and report already-finalized state without double release.

Commit boundary: complete context lifecycle, operation wait, and ownership state machine.

### PH5-C2. Context Lifecycle Phase Gate

Design section: [Initialization and Finalization](detailed-design.md#initialization-and-finalization).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_05_context_lifecycle.py` using dummy compilers and dummy QRI.
- [ ] Exercise successful ordered initialization and finalization, illegal-state calls, each initialization failure boundary, reverse unwind, successful wait, timeout with exact outstanding IDs, repeated cleanup, and external ownership preservation.
- [ ] Assert finalization never calls QPM cancellation implicitly, never claims timed-out operations completed, and never shuts down the dummy shared QPM.
- [ ] Create `docs/phase-tests/phase-05-context-lifecycle.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_05_context_lifecycle.py` and record the passing evidence.

Commit boundary: Phase 5 executable lifecycle contract and its test document.

## Phase 6

### PH6-C1. Compatibility Map Construction

Design section: [Compatibility Mapping](detailed-design.md#compatibility-mapping).

Requirements: `LQ-COMP-004`, `LQ-COMP-008`, `LQ-FMT-003`, `LQ-FMT-004`, `LQ-CON-002`.

Checklist:

- [ ] Cross every compiler output with every allocated device input by exact representation and exact shared version.
- [ ] Create one available definition per compiler input, compiler output, device input, and negotiated output version.
- [ ] Index definitions by exact compiler input/version and device.
- [ ] Construct opaque definitions separately with compiler and format fields absent.
- [ ] Derive deterministic route IDs and map generation from every binding named by the design.
- [ ] Keep map data immutable and structural; do not run program analysis, capacity checks, ranking, or selection here.
- [ ] Rebuild only on initialization or explicit refresh.
- [ ] Retain old definitions and bindings for routes opened before refresh.

Commit boundary: deterministic compatibility map construction.

### PH6-C2. Compatibility Mapping Phase Gate

Design section: [Compatibility Mapping](detailed-design.md#compatibility-mapping).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_06_compatibility_mapping.py` using both dummy compilers and both allocated dummy devices.
- [ ] Prove the exact expected route cross-product, version intersection, input/device indexes, opaque definitions, deterministic IDs, and generation changes.
- [ ] Prove mismatched and unreported versions create no route and the unrelated device creates no route.
- [ ] Prove refresh leaves a previously opened definition unchanged.
- [ ] Create `docs/phase-tests/phase-06-compatibility-mapping.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_06_compatibility_mapping.py` and record the passing evidence.

Commit boundary: Phase 6 executable mapping contract and its test document.

## Phase 7

### PH7-C1. Program Import and Route Query

Design sections: [Program Representation](detailed-design.md#program-representation) and [Route Selection](detailed-design.md#route-selection).

Requirements: `LQ-PRG-001` through `LQ-PRG-005`, `LQ-PRG-007`, `LQ-COMP-005` through `LQ-COMP-007`, `LQ-SEL-001`, `LQ-SEL-002`, `LQ-SEL-008`, `LQ-DEMO-006`.

Checklist:

- [ ] Implement explicit representation validation against the payload.
- [ ] Identify SDK objects by exact registered type and installed SDK version.
- [ ] Run serialized recognizers, deduplicate identical matches, and reject unknown or conflicting matches.
- [ ] Require explicit `import_opaque()` intent; never fall back from failed recognition to opaque.
- [ ] Preserve caller requirements and allow compiler analysis only to fill missing information, never weaken constraints.
- [ ] Filter structural routes by exact program representation/version.
- [ ] Run analysis, compatibility, target completeness, feature, and usable-capacity checks per candidate.
- [ ] Apply every `RouteConstraints` field as a filter and report no eligible route before submission.
- [ ] Return all eligible definitions in deterministic route-ID order without ranking or choosing a winner.
- [ ] Return structured rejection stages, reason codes, and diagnostics when `explain=True`.

Commit boundary: program import and inspectable route query.

### PH7-C2. Route Opening and Binding

Design sections: [Program Route](detailed-design.md#program-route) and [Route Selection](detailed-design.md#route-selection).

Checklist:

- [ ] Validate that an available definition belongs to the context and current compatibility generation.
- [ ] Create a fresh instance ID on every open while preserving the definition's route ID.
- [ ] Bind the exact compiler callbacks/state, immutable device definition, private access context, negotiated format/version, and QRI dispatch once.
- [ ] Require opaque programs to use opaque definitions with no compiler callbacks.
- [ ] Keep opened routes stable across resource refresh.
- [ ] Reject a route, program, or constraint object owned by another context where ownership applies.

Commit boundary: selected route instance creation and fixed dispatch bindings.

### PH7-C3. Route Selection Phase Gate

Design section: [Route Selection](detailed-design.md#route-selection).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_07_route_selection.py`.
- [ ] Exercise SDK, serialized, explicit, unknown, ambiguous, and opaque imports.
- [ ] Exercise every constraint field, capacity rejection, compiler rejection, explanation output, deterministic ordering, no-ranking behavior, context/generation rejection, repeated open, and refresh stability.
- [ ] Assert dummy callback logs show analysis/compatibility only for applicable candidates and no compiler calls for opaque input.
- [ ] Create `docs/phase-tests/phase-07-route-selection.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_07_route_selection.py` and record the passing evidence.

Commit boundary: Phase 7 executable route-selection contract and its test document.

## Phase 8

### PH8-C1. Compilation Dispatch

Design section: [Compilation and Execution](detailed-design.md#compilation-and-execution).

Requirements: `LQ-PRG-004`, `LQ-COMP-008`, `LQ-COMP-009`, `LQ-TASK-001`, `LQ-FMT-006`.

Checklist:

- [ ] Dispatch `compile(route, program)` through the compiler callback and state already bound to the application-selected route.
- [ ] Pass the program, selected device context, and exact negotiated output representation/version.
- [ ] Return `CompilerResult` unchanged, including payload, representation, snapshots, layout, parameter metadata, provenance, and diagnostics.
- [ ] Perform no QRI submission, capability discovery, route ranking/selection, or compatibility analysis in `compile()`.
- [ ] Reject compilation through an opaque route.
- [ ] Return compilation failures synchronously as compilation-stage `LibquantumError` before any QRI operation exists.

Commit boundary: compilation-only behavior through the opened route.

### PH8-C2. Submission and Operation Control

Design section: [Compilation and Execution](detailed-design.md#compilation-and-execution).

Requirements: `LQ-TASK-002` through `LQ-TASK-005`, `LQ-QFW-002`, `LQ-QFW-005`, `LQ-ADAPT-006`, `LQ-ADAPT-008`, `LQ-CON-002`.

Checklist:

- [ ] Implement `run(route, executable, completion_queue, execution_options)` as QRI submission only.
- [ ] Require the queue to belong to the same context and opened route.
- [ ] Prepare `CompilerResult.payload` through the existing QFw payload mechanism; forward an opaque `Program.payload` unchanged.
- [ ] Invoke no compiler callback and perform no route query or selection inside `run()`.
- [ ] Call the route-bound QPM `async_run` with the reservation, queue registration, and exact execution options.
- [ ] Extract and validate QRI `cid`, store the private operation record, and return that string as `op_id`.
- [ ] Create no record when submission fails and expose no QRI response mapping.
- [ ] Implement `status(op_id)` through the tracked QPM's `task_status` and return its mapping unchanged.
- [ ] Implement `cancel(op_id)` through the tracked QPM's `cancel_task`; preserve requested, unsupported, uncertain, and confirmed outcomes.
- [ ] Support several independently outstanding operation IDs.

Commit boundary: submission-only `run()` and QPM-backed operation control.

### PH8-C3. Compilation and Execution Phase Gate

Design section: [Compilation and Execution](detailed-design.md#compilation-and-execution).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_08_compilation_execution.py`.
- [ ] Prove exact compiler callback propagation and unchanged compiler result identity without QRI submission.
- [ ] Prove `run()` submits compiled and opaque payloads through the bound route, invokes no compiler, and returns only QRI `cid` as `op_id`.
- [ ] Prove accepted operations are tracked with the correct route, queue, QPM, and reservation.
- [ ] Prove `status(op_id)` and `cancel(op_id)` call the owning dummy QPM and reject unknown IDs.
- [ ] Prove cancellation is not confirmed until the dummy QPM reports a terminal cancelled outcome.
- [ ] Exercise compiler, immediate submission, status, and cancellation failure translation boundaries.
- [ ] Create `docs/phase-tests/phase-08-compilation-execution.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_08_compilation_execution.py` and record the passing evidence.

Commit boundary: Phase 8 executable compile, submit, status, and cancellation contract.

## Phase 9

### PH9-C1. Completion Queue Delivery

Design section: [Completion Queues](detailed-design.md#completion-queues).

Requirements: `LQ-CQ-001` through `LQ-CQ-007`, `LQ-RSLT-001`, `LQ-TASK-005`.

Checklist:

- [ ] Create one `BaseEventAPI` endpoint per queue and register it through the selected route's QPM.
- [ ] Bind the queue to its route and store the registration used by `run()` and cleanup.
- [ ] Implement `fileno()` and nonconsuming `wait(timeout)`; timeout returns false and changes no QPM status.
- [ ] Implement `get()` to drain events, expose QRI `cid` as `Completion.op_id`, and preserve successful payloads unchanged.
- [ ] Mark the matching private operation record terminal when its event is consumed.
- [ ] Store an operation failure only in that operation's completion; raise endpoint failures at the queue boundary.
- [ ] Support multiple queues and several operations sharing one queue without a central redistribution layer.
- [ ] Do not add application correlation metadata or a second deduplication/reconciliation system.
- [ ] Unregister and release local endpoint resources idempotently; closing a queue does not cancel operations.

Commit boundary: direct route-bound completion delivery and operation completion tracking.

### PH9-C2. Completion Queue Phase Gate

Design section: [Completion Queues](detailed-design.md#completion-queues).

Checklist:

- [ ] Extend dummy event/QPM fixtures to inject successful events, operation failures, batch mixtures, queue transport failures, and timeouts.
- [ ] Create `tests/phase_tests/test_phase_09_completion_queues.py`.
- [ ] Prove registration routing, nonconsuming waits, timeout behavior, batch draining, `op_id`/QRI `cid` identity, payload identity, per-operation isolation and tracked-operation terminal marking, multiple queues, shared queues, and idempotent close/finalize cleanup.
- [ ] Create `docs/phase-tests/phase-09-completion-queues.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_09_completion_queues.py` and record the passing evidence.

Commit boundary: Phase 9 executable completion contract and its test document.

## Phase 10

### PH10-C1. Error Translation and Provenance

Design section: [Errors, Results, and Provenance](detailed-design.md#errors-results-and-provenance).

Requirements: `LQ-DEV-003`, `LQ-OBS-001`, `LQ-OBS-002`, `LQ-OBS-004`, `LQ-OBS-005`.

Checklist:

- [ ] Implement `ErrorStage`, immutable `ErrorDetail`, and `LibquantumError`.
- [ ] Define stable libquantum codes for recognized QFw/DEFw categories and one safe underlying-service fallback per applicable stage.
- [ ] Translate synchronous discovery, registration, submission, status, cancellation, wait, retrieval, cleanup, finalization, and compiler failures at their public boundary.
- [ ] Retain route, constraints, and `op_id` only when available.
- [ ] Exclude credentials, server tracebacks, service objects, and original exception types from public detail.
- [ ] Keep original exceptions private for diagnostics where supported.
- [ ] Preserve compiler result and successful QRI payloads unchanged.
- [ ] Keep unavailable compiler/provider/timing/trace fields absent.

Commit boundary: one consistent public failure and provenance contract.

### PH10-C2. Errors and Results Phase Gate

Design section: [Errors, Results, and Provenance](detailed-design.md#errors-results-and-provenance).

Checklist:

- [ ] Create `tests/phase_tests/test_phase_10_errors_results.py`.
- [ ] Inject a failure at every public boundary, including QPM status and cancellation and verify stage, stable code, safe message, available correlation, and absence of service internals.
- [ ] Verify one failed operation does not hide successful batch peers.
- [ ] Verify compiler and QRI success values and provenance retain identity and no missing field is fabricated.
- [ ] Create `docs/phase-tests/phase-10-errors-results.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_10_errors_results.py` and record the passing evidence.

Commit boundary: Phase 10 executable cross-cutting error/result contract and its test document.

## Phase 11

### PH11-C1. Isolated End-to-End Workflow

Design sections: [Purpose](detailed-design.md#purpose) and [Application Workflow](detailed-design.md#application-workflow).

Requirements: `LQ-INT-001` through `LQ-INT-003`, `LQ-APP-001`, `LQ-APP-002`, `LQ-COMP-009`, `LQ-DEMOINT-004`, `LQ-DEMOINT-005`.

Checklist:

- [ ] Create `tests/phase_tests/test_phase_11_isolated_workflow.py` using only dummy compiler, QRI/QPM, devices, and events.
- [ ] Execute initialize, program import, route query, application route choice/open, queue creation, separate compilation, several `run()` calls, status, cancellation, successful and failed completions, and finalization.
- [ ] Verify every `run()` returns only the QRI `cid` as `op_id` and creates the expected internal operation record.
- [ ] Execute opaque import and `run()` and prove unchanged payload identity and zero compiler calls.
- [ ] Exercise at least two application-selected compiler/device routes and two completion queues.
- [ ] Exercise finalization success and timeout reporting with exact outstanding operation IDs.
- [ ] Prove the workflow imports no SDK execution backend and needs no external service or credential.
- [ ] Create `docs/phase-tests/phase-11-isolated-workflow.md`.
- [ ] Run `python -m pytest -q tests/phase_tests/test_phase_11_isolated_workflow.py` and retain the callback/submission/event trace.

Commit boundary: complete isolated libquantum lifecycle proof.

### PH11-C2. Traceability and Drift Audit

Design section: [Future Requirements](detailed-design.md#future-requirements).

Checklist:

- [ ] Build a traceability table from every requirement ID to its design section, implementation unit, and phase test evidence.
- [ ] Mark requirements listed by the design as future even when the requirements artifact currently labels them SC26; do not claim implementation evidence for them.
- [ ] Identify every requirements/design scope conflict as an artifact issue requiring operator resolution.
- [ ] Verify every non-future design statement has at least one implementation task and every public behavior has phase-gate evidence.
- [ ] Verify no checklist task introduces graph/executor, automatic policy ranking, compiler configuration, result normalization, runtime plugin installation, or deployed hardware behavior absent from the prototype design.
- [ ] Re-run all phase modules individually using the exact command in each companion document.
- [ ] Create `docs/phase-tests/phase-11-traceability-review.md` with the coverage table, commands, results, known gaps, and artifact discrepancies.

Commit boundary: auditable plan/design/requirements coverage without overstating prototype scope.

## Phase 12

### PH12-C1. Managed QFw Integration Gate

Design sections: [Device Discovery Through QRI](detailed-design.md#device-discovery-through-qri), [Initialization and Finalization](detailed-design.md#initialization-and-finalization), [Compilation and Execution](detailed-design.md#compilation-and-execution), and [Completion Queues](detailed-design.md#completion-queues).

This deployment gate follows the isolated proof and is runnable only in a frozen QFw environment supplied by the operator. It validates the designed managed-QRI boundary; broader SC26 qualification requirements remain deferred by the design's Future Requirements section.

Checklist:

- [ ] Create `docs/phase-tests/phase-12-qfw-integration.md` with exact frozen runtime, allocation, reservation, QPM, device, compiler, credential, and cleanup prerequisites.
- [ ] Create `tests/phase_tests/test_phase_12_qfw_integration.py` with an explicit integration marker and safe skip when prerequisites are absent. A skip confirms only that the local environment is isolated; it is not qualification evidence.
- [ ] Replace dummy runtime/device/compiler inputs one boundary at a time while retaining the same public application flow.
- [ ] Verify allocation-scoped discovery, route binding, payload submission, `op_id` identity, QPM status/cancellation, terminal delivery, and listener cleanup against deployed QFw.
- [ ] Record deployed component revisions and clearly distinguish simulator/mock evidence from physical-device evidence.
- [ ] Run `python -m pytest -q -m qfw_integration tests/phase_tests/test_phase_12_qfw_integration.py` in the frozen environment and retain logs without credentials or service internals.
- [ ] Report unsupported combinations and design gaps explicitly; do not add automatic fallback or a dependency on QFw Backend.

Commit boundary: deployed managed-QRI integration evidence limited to the current detailed design.
