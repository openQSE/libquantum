# libquantum Requirements

**Status:** draft

## Goal

> **An application can construct a quantum program with one SDK, compile it with another toolchain, and execute it through QRI without submitting through an SDK execution backend.**

libquantum provides public application APIs for resource discovery, route selection, compilation, execution, task observation, and result delivery. Graph construction and its executor are optional convenience layers outside the SC26 milestone and use the same public APIs as applications. The SDK used to express a program, the compiler used to transform it, and the hardware used to execute it can vary independently across explicitly supported combinations.

An application owns algorithmic control flow, parameter updates, and data binding. Programs and their compiler/runtime integrations own qubit operations, parameter binding, measurement ordering, and result interpretation. libquantum uses program-level capability and resource metadata to select routes, submit whole programs, and deliver program-level results. Application code controls program ordering through the public APIs; a future graph executor can provide sequential or concurrent ordering as a convenience. Independent tasks may progress concurrently. The library does not infer an algorithm from a circuit or invent workflow logic.

QRI means the quantum resource interface boundary. In the SC26 deployment, libquantum reaches that boundary through QFw's managed QPM execution and telemetry APIs; QRMI, QDMI, and provider implementations remain below it. Compiler target descriptions and internal provider SDK helpers are permitted. The application must not need an SDK backend, provider, job, or device plugin to submit through libquantum.

This document defines observable requirements. Exact API signatures, callback table layouts, graph storage, scheduling algorithms, and package layout belong in the detailed design. All records are draft. Every identified requirement belongs to the overall libquantum scope; the SC26 column selects the subset required for the demonstration. The upstream review is informative evidence rather than a claim that proposed work is deployed.

## Target Users

- Application developers who construct programs in supported quantum SDKs and want a common compilation and execution interface.
- Hybrid application developers who control ordered or concurrent quantum program submissions in application code and receive results through one or more completion queues.
- Compiler integrators who expose a compiler's supported inputs, target capabilities, configuration, and outputs.
- QFw integrators and demo operators who provide allocations, reservations, QPM services, device descriptions, credentials, and a reproducible software environment.

## Workflows

The SC26 workflow begins after the launcher or resource manager has made resources available. Route discovery, compilation, and QRI submission are separate application operations.

1. Initialize libquantum using the active runtime and allocation context.
2. During startup, register configured compiler integrations, discover allocated devices through QRI, query compiler/device format support and versions, and enumerate matching compiler/output/device routes.
3. Construct or import a supported program, or explicitly supply an opaque blob.
4. Query eligible routes and apply compiler, device, format, version, and capacity constraints. The application chooses an `AvailableRoute` and opens it as a `ProgramRoute`.
5. For a recognized program, call the separate compilation API with the opened route and retain its compiler result. Opaque programs bypass compilation.
6. Create a completion queue for the opened route and call `run()` with the compiler result or opaque program. `run()` only submits through the route's QRI/QPM binding and returns the QRI `cid` as the operation ID.
7. Use the operation ID to query QPM-authoritative status or request QPM cancellation. Terminal QRI events arrive on the selected completion queue and carry the same operation ID.
8. Finalize libquantum after resolving outstanding operations. Finalization waits up to its timeout; if operations remain, it reports their IDs and returns without claiming completion or cancellation.

Public route queries and compilation remain independently accessible. libquantum filters and explains eligible routes but does not rank them or choose one for the application. Compiler configuration and automatic selection policies are future scope. Direct submission of a stored compatible compiler result uses `run()`; accepting arbitrary previously compiled artifacts remains future scope under `LQ-PRG-009`.

```mermaid
flowchart LR
    SDK["SDK circuit or serialized payload"] --> Import["import_program()"]
    Import --> Query["list_routes()"]
    Query --> Choose["Application chooses route"]
    Choose --> Open["open_route()"]
    Open --> Compile["compile(route, program)"]
    Compile --> Run["run(route, compiler_result, queue)"]
    Blob["Explicit opaque program"] --> Run
    Run -->|"op_id = QRI cid"| QRI["QRI/QPM"]
    QRI --> Queue["Selected completion queue"]
```

The application controls route choice, compilation timing, submission ordering, and result interpretation.

## Required Behavior

Each category contains individually referenceable requirements. **SC26 = Yes** means required for the SC26 demonstration. **SC26 = No** means part of the overall requirements but outside the SC26 milestone; it does not mean rejected or optional for the overall product.

Identifiers use `LQ-<category>-<number>`, with consecutive three-digit integers starting at `001` in each category. After adding, removing, or reordering requirements, renumber the affected category and update every reference. Keep the numbering gap-free and use whole numbers without decimal or subrequirement suffixes.

Normative statements use **shall**. Functional requirements describe what libquantum does or exposes; application-supplied programs, routing metadata, submission options, and allocation context are inputs to that behavior. Informative workflow descriptions, acceptance evidence, upstream findings, and open decisions provide context and do not add requirements without an ID. All requirement tables have the same three columns.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-int-001"></a>`LQ-INT-001` | libquantum shall provide a common application interface that allows a supported SDK-origin quantum program to be compiled with another supported toolchain and executed through QRI without application submission through an SDK execution backend. | Yes |
| <a id="lq-int-002"></a>`LQ-INT-002` | libquantum shall expose resource discovery, route selection, compilation, and execution as separate public operations usable directly by applications independently of graph or executor objects, including compilation without device submission. | Yes |
| <a id="lq-int-003"></a>`LQ-INT-003` | libquantum shall provide consistent public program, task, and completion interfaces across supported SDK, compiler, and device combinations. | Yes |

### Initialization

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-init-001"></a>`LQ-INIT-001` | libquantum shall initialize its context from the active QFw runtime. | Yes |
| <a id="lq-init-002"></a>`LQ-INIT-002` | libquantum shall establish resource discovery, compiler registration, compatibility mapping, task submission, and completion-notification capabilities during initialization. Connections and compiler resources used only for later compilation or execution may be opened when first needed. | Yes |
| <a id="lq-init-003"></a>`LQ-INIT-003` | If initialization fails, libquantum shall report the failure reason and identify the missing or invalid configuration, unavailable runtime information, or unreachable service responsible. | Yes |
| <a id="lq-init-004"></a>`LQ-INIT-004` | libquantum shall preserve the relationship between the allocation, selected device, service identity, and reservation during initialization and subsequent operations. | Yes |

### Finalization

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-fin-001"></a>`LQ-FIN-001` | Finalization shall stop acceptance of new work, wait for libquantum-tracked operations to reach a terminal QPM state up to a caller-specified timeout, unregister completion listeners, and release library-owned connections, queues, and compiler resources. | Yes |
| <a id="lq-fin-002"></a>`LQ-FIN-002` | Normal finalization shall return after all tracked operations complete. It shall not implicitly cancel an operation; cancellation remains an explicit application request. | Yes |
| <a id="lq-fin-003"></a>`LQ-FIN-003` | If finalization times out, libquantum shall report the outstanding operation IDs and return without claiming that they completed or were cancelled. | Yes |
| <a id="lq-fin-004"></a>`LQ-FIN-004` | Cleanup shall be safe to call again and shall not release an externally owned allocation or shut down a shared QPM. | Yes |
| <a id="lq-fin-005"></a>`LQ-FIN-005` | Unsupported QPM cancellation or a disconnected service shall be reported without claiming confirmed cancellation. | Yes |

### Allocation Device Discovery

The allocation establishes the authorized device set, and the QRI supplies the authoritative device information.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-res-001"></a>`LQ-RES-001` | libquantum shall discover only the devices belonging to a given allocation through the QRI. | Yes |
| <a id="lq-res-002"></a>`LQ-RES-002` | libquantum shall maintain an up-to-date local mirror of device information reported by the QRI for that allocation. | Yes |

### Unified Device Representation

The QRI supplies the device description and defines the meaning of its fields. The compiler integration retrieves the target information its compilation passes require and adapts it for the compiler. libquantum supplies access information for the selected device and retains the compilation provenance reported by the integration. For SC26, this access uses QFw's managed QRI telemetry interfaces.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-dev-001"></a>`LQ-DEV-001` | libquantum shall expose QRI-reported device information through a common device representation. | Yes |
| <a id="lq-dev-002"></a>`LQ-DEV-002` | libquantum shall provide the selected compiler integration with the QRI endpoint and device context required to retrieve target information for compilation. | Yes |
| <a id="lq-dev-003"></a>`LQ-DEV-003` | libquantum shall record the source and target snapshot reported by the compiler integration for each compilation. | Yes |
| <a id="lq-dev-004"></a>`LQ-DEV-004` | libquantum shall allow multiple execution routes to reference the same device representation. | Yes |

### Unified Program Representation

Inputs may be supported SDK circuit objects, recognized serialized programs, or opaque blobs that libquantum cannot identify. Recognized inputs carry SDK/type or format/version information for compiler selection. The receiving backend interprets and validates opaque payloads. Program resource requirements come from application metadata or compiler callbacks; compiler/runtime integrations interpret program contents and their execution semantics. Direct managed QRI submission and task/completion delivery apply to both paths; graph ordering is an optional future convenience.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-prg-001"></a>`LQ-PRG-001` | libquantum shall accept supported SDK circuit objects as program inputs. | Yes |
| <a id="lq-prg-002"></a>`LQ-PRG-002` | libquantum shall accept unidentified inputs as opaque blobs and forward their payloads unchanged to the configured execution backend through QRI, without parsing, converting, or compiling those payloads. | Yes |
| <a id="lq-prg-003"></a>`LQ-PRG-003` | libquantum shall retain each recognized program's object or payload and its SDK/object type or serialized format and applicable version, using application-supplied metadata or identification of a supported input. | Yes |
| <a id="lq-prg-004"></a>`LQ-PRG-004` | libquantum shall hand accepted program objects or payloads and application-supplied inputs to the selected compiler/runtime integration through its declared interface. | Yes |
| <a id="lq-prg-005"></a>`LQ-PRG-005` | The library shall accept several supported representations without forcing every program through a single universal circuit format. | Yes |
| <a id="lq-prg-006"></a>`LQ-PRG-006` | For recognized program representations, libquantum shall propagate compiler/runtime integration diagnostics about unsupported constructs or conversions that would lose required semantics before affected work is submitted. | No |
| <a id="lq-prg-007"></a>`LQ-PRG-007` | The original program shall remain available when compilation fails or another supported compiler is requested. | Yes |
| <a id="lq-prg-008"></a>`LQ-PRG-008` | For recognized inputs, the program contract shall distinguish a serialized circuit from a complete optimizer or arbitrary hybrid application; algorithmic logic shall remain with the application or explicitly supplied workload code. | No |
| <a id="lq-prg-009"></a>`LQ-PRG-009` | libquantum shall support direct execution of a compatible previously compiled artifact through an application-facing API, using its selected route and required runtime inputs. | No |

### Compiler Registration and Support

Compiler integrations register callbacks with libquantum. During startup, libquantum queries exact input and output representations and versions and combines them with QRI-reported device input support. Compatibility callbacks filter routes for a program; compilation callbacks compile through the application-selected route.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-cmp-001"></a>`LQ-CMP-001` | libquantum shall provide a registration mechanism through which compiler integrations supply callbacks for input and output format/version queries, other capability queries, compatibility checks, compilation, initialization, and cleanup. | Yes |
| <a id="lq-cmp-002"></a>`LQ-CMP-002` | libquantum shall obtain each compiler's identity, version, availability, accepted input representations and versions classified as native or internally converted, supported features, target constraints, and output representations and versions through registered callbacks. | Yes |
| <a id="lq-cmp-003"></a>`LQ-CMP-003` | libquantum shall invoke registered compatibility and compilation callbacks with the program, selected device context, and requested output representation/version, and receive compilation outputs and diagnostics through a common contract. | Yes |
| <a id="lq-cmp-004"></a>`LQ-CMP-004` | libquantum shall expose every eligible registered compiler/device route after applying capability, compatibility, and application-supplied constraints; the application shall choose the route. | Yes |
| <a id="lq-cmp-005"></a>`LQ-CMP-005` | libquantum shall initialize and release compiler integrations through their registered lifecycle callbacks. The SC26 implementation may use a static set of installed Python integrations. | Yes |

Per-route compiler configuration and pass pipelines are future scope.

### Compatibility Mapping

During startup, libquantum enumerates all reported compiler/output/device routes by matching compiler output formats and versions to allocated-device input formats and versions. Each route identifies a compiler integration, an output format/version, and a device. The compiler's accepted input formats and versions index the relevant routes; several accepted inputs may refer to the same route.

Compiler integrations report native input formats and versions separately from additional inputs they accept through internal conversion. This classification is compiler capability metadata used for selection preferences. The integration handles input conversion within its compilation operation, and existing payload mechanisms handle output serialization. Route enumeration uses the reported input/output capabilities and device format matches.

When a program arrives, libquantum filters the startup route set by all applicable program requirements and submission options before ranking. Native and internally converted inputs both participate in input matching; native support is a preference among eligible routes. QRI supplies usable device capacity, while application metadata and registered compiler callbacks report the program's requirements.

A 50-qubit program whose only compatible compiler/output route targets a 20-qubit device has no eligible route and fails selection before device submission. A larger device is usable when a compatible route to it exists.

Opaque blobs follow `LQ-PRG-002`; their payload interpretation and acceptance remain with the receiving backend.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-comp-001"></a>`LQ-COMP-001` | libquantum shall query registered compiler input-capability callbacks during startup for native input formats and versions, and for additional input formats and versions accepted through conversion handled internally by the integration. | Yes |
| <a id="lq-comp-002"></a>`LQ-COMP-002` | libquantum shall query registered compiler output-capability callbacks during startup for the output formats and versions the integration can provide for device submission. | Yes |
| <a id="lq-comp-003"></a>`LQ-COMP-003` | libquantum shall query each allocated device through QRI during startup for its accepted input formats and supported versions. | Yes |
| <a id="lq-comp-004"></a>`LQ-COMP-004` | libquantum shall enumerate and retain all startup routes by matching each registered compiler's output formats and versions to every allocated device's accepted input formats and versions, and index those routes by the compiler's supported input formats and versions. | Yes |
| <a id="lq-comp-005"></a>`LQ-COMP-005` | libquantum shall validate each candidate route for a submitted program and supplied options using registered compiler compatibility callbacks and the execution requirements reported through QRI. | Yes |
| <a id="lq-comp-006"></a>`LQ-COMP-006` | libquantum shall filter startup routes using the submitted input format/version, compiler compatibility, explicit selections, and required qubit capacity, and report a selection failure before device submission if no eligible route remains. | Yes |
| <a id="lq-comp-007"></a>`LQ-COMP-007` | libquantum shall provide a query API that, on explicit caller request, returns the startup compatibility mapping or compatibility details for a requested program/compiler/device combination as structured data, including reasons for missing or rejected routes. | Yes |
| <a id="lq-comp-008"></a>`LQ-COMP-008` | libquantum shall submit compiler results using the route's selected output format/version and the existing payload serialization mechanism, and verify that the resulting payload matches the submission contract. | Yes |
| <a id="lq-comp-009"></a>`LQ-COMP-009` | libquantum shall distinguish successful compilation, successful execution submission, and completion reported by the execution runtime as separate outcomes. | Yes |

### Application Route Selection

libquantum determines route eligibility; the application chooses among the returned routes. Explicit compiler, device, representation, version, and capacity values are filters rather than ranking preferences.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-sel-001"></a>`LQ-SEL-001` | libquantum shall provide an application interface to constrain eligible routes by registered compiler, allocated device, or both. | Yes |
| <a id="lq-sel-002"></a>`LQ-SEL-002` | libquantum shall apply every explicit compiler and device selection as a strict constraint and return incompatibility when no route satisfies it. | Yes |
| <a id="lq-sel-003"></a>`LQ-SEL-003` | libquantum shall provide a public route-query API that uses the submitted representation/version and startup mapping to return every compatible route; the application shall choose one and pass it to `open_route()`. | Yes |
| <a id="lq-sel-004"></a>`LQ-SEL-004` | libquantum shall provide an application interface to configure enabled device and compiler selection policies and their preference order. | No |
| <a id="lq-sel-005"></a>`LQ-SEL-005` | libquantum shall provide policy-decision explanations after automatic selection is added. | No |
| <a id="lq-sel-006"></a>`LQ-SEL-006` | libquantum shall recheck target and usable-capacity constraints after compilation, including added qubits and reservation limits. | No |
| <a id="lq-sel-007"></a>`LQ-SEL-007` | Automatic selection shall not silently change hardware, compiler, or execution path after remote acceptance or an uncertain submission outcome. | No |
| <a id="lq-sel-008"></a>`LQ-SEL-008` | libquantum shall return eligible routes in deterministic `route_id` order; that order shall not express a preference. | Yes |

### Future Selection Policies

Automatic selection policies are future scope. The SC26 prototype exposes eligibility metadata, native/converted classification, capacity, and deterministic route ordering so the application can implement its own choice.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-pol-001"></a>`LQ-POL-001` | libquantum shall provide a device-selection policy that prefers the smallest eligible allocated device with sufficient usable qubit capacity for the program. The SC26 default shall apply this device preference before compiler preferences. | No |
| <a id="lq-pol-002"></a>`LQ-POL-002` | libquantum shall provide a compiler-selection policy that ranks compatible registered compilers by an application-configurable preference order, including orders specific to a device. | No |
| <a id="lq-pol-003"></a>`LQ-POL-003` | libquantum shall provide a compiler-selection policy that uses compiler capability reports to prefer native support for the submitted representation and version over acceptance through internal conversion, among eligible routes. The SC26 default shall apply this preference before compiler priority. | No |
| <a id="lq-pol-004"></a>`LQ-POL-004` | libquantum shall provide a policy that ranks compatible compiler/device pairs by a configured compiled-circuit metric, such as two-qubit operation count or circuit depth, using compiler estimates or trial compilation outputs. | No |
| <a id="lq-pol-005"></a>`LQ-POL-005` | libquantum shall provide a policy that prefers compatible compiler/device pairs with the best execution-quality estimates reported by compiler integrations using available QRI target and calibration information. | No |
| <a id="lq-pol-006"></a>`LQ-POL-006` | libquantum shall provide a policy that prefers the lowest estimated total turnaround time across compatible routes, using compilation estimates from compiler integrations and waiting and execution estimates from QRI. | No |
| <a id="lq-pol-007"></a>`LQ-POL-007` | libquantum shall provide an affinity policy that prefers a previously selected compatible compiler/device pair for related tasks or successive iterations identified by the application. | No |
| <a id="lq-pol-008"></a>`LQ-POL-008` | libquantum shall provide a policy that distributes eligible independent graph nodes across compatible allocated devices using QRI-reported resource information, while preserving graph dependencies and QFw's admission and scheduling responsibilities. | No |

### Graph Construction

Graph construction is an optional convenience outside SC26. Initial graph support collects quantum-program nodes and their metadata under a single sequential or concurrent mode. Classical task nodes are a later extension. Routing metadata identifies a selected route or constraints for the public route selection API; it may specify a destination for opaque input.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-grf-001"></a>`LQ-GRF-001` | libquantum shall provide APIs to create a graph and add quantum programs as nodes. | No |
| <a id="lq-grf-002"></a>`LQ-GRF-002` | libquantum shall provide a graph-wide ordering mode with two choices: sequential or concurrent. | No |
| <a id="lq-grf-003"></a>`LQ-GRF-003` | libquantum shall attach application-supplied routing metadata, compilation/execution options, completion-queue selection, and correlation data to each quantum-program node. | No |
| <a id="lq-grf-004"></a>`LQ-GRF-004` | libquantum shall provide APIs to add explicitly supplied classical tasks as graph nodes as a later extension. | No |

### Graph Executor

The graph executor is an optional convenience outside SC26. It traverses a constructed graph and invokes the same public operations that applications can call directly. Initial nodes contain quantum programs; later classical nodes extend the node model. Program semantics and application data flow remain with the supplied programs and application code.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-exe-001"></a>`LQ-EXE-001` | libquantum shall provide an executor API to run a constructed graph in its selected sequential or concurrent mode. | No |
| <a id="lq-exe-002"></a>`LQ-EXE-002` | libquantum shall execute quantum-program nodes using the same public route selection, compilation, execution, task, and completion APIs available to applications, as required by each program's representation and routing metadata. | No |
| <a id="lq-exe-003"></a>`LQ-EXE-003` | libquantum shall process sequential graph nodes in insertion order, waiting for successful completion of one program before processing the next. | No |
| <a id="lq-exe-004"></a>`LQ-EXE-004` | libquantum shall submit independent nodes in a concurrent graph without waiting for other nodes to complete, subject to the public execution API's admission limits. | No |
| <a id="lq-exe-005"></a>`LQ-EXE-005` | libquantum shall provide graph status and node outcomes through the executor API, marking later sequential nodes as skipped after failure or cancellation while allowing independent concurrent nodes to finish. | No |

### Compilation and Execution APIs

Compilation and QRI submission are separate public operations. `compile()` returns a compiler result without submitting it. `run()` submits an already compiled result, or an explicitly opaque program, through an opened route and returns the QRI `cid` as the operation ID.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-task-001"></a>`LQ-TASK-001` | libquantum shall provide a compilation API that accepts a supported program and an application-selected route and returns the compiler result or a structured error without submitting work to QPM. | Yes |
| <a id="lq-task-002"></a>`LQ-TASK-002` | libquantum shall provide an asynchronous `run()` API that accepts an opened route, a compiler result or opaque program, a completion queue, and execution options; submits through QRI; tracks the accepted operation internally; and returns the QRI `cid` as the operation ID. `run()` shall not select a route or compile a program. | Yes |
| <a id="lq-task-003"></a>`LQ-TASK-003` | libquantum shall provide `status(op_id)` and query the owning QPM's `task_status` operation using the tracked route and reservation context. | Yes |
| <a id="lq-task-004"></a>`LQ-TASK-004` | libquantum shall provide `cancel(op_id)` and forward the request to the owning QPM's `cancel_task` operation. Confirmed cancellation shall be reported through the selected completion queue. | Yes |
| <a id="lq-task-005"></a>`LQ-TASK-005` | libquantum shall support multiple independent outstanding operations, subject to QPM admission and provider limits. | Yes |

### Completion Queues

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-cq-001"></a>`LQ-CQ-001` | libquantum shall create multiple completion queues, associate each submitted operation with its selected queue, and support sharing a queue among multiple independent operations. | Yes |
| <a id="lq-cq-002"></a>`LQ-CQ-002` | libquantum shall expose the terminal QRI/QPM completion for each accepted operation, including execution failure, execution timeout, or confirmed cancellation. | Yes |
| <a id="lq-cq-003"></a>`LQ-CQ-003` | Each completion shall contain the operation ID derived from QRI `cid` and either the unchanged result payload or a structured error. | Yes |
| <a id="lq-cq-004"></a>`LQ-CQ-004` | libquantum shall mark a tracked operation complete when its terminal QRI/QPM event is consumed from the operation's selected queue. It shall not create a central event redistribution or reconciliation layer. | Yes |
| <a id="lq-cq-005"></a>`LQ-CQ-005` | The library shall preserve supported QFw completion polling and event semantics and shall not consume another logical client's result. | Yes |
| <a id="lq-cq-006"></a>`LQ-CQ-006` | libquantum shall provide operations to wait for and consume completions and retrieve their result payloads. A caller-side wait timeout shall return without publishing a completion or changing QPM task status. | Yes |
| <a id="lq-cq-007"></a>`LQ-CQ-007` | libquantum shall unregister its completion listeners during cleanup. | Yes |

### Program Results and Delivery

QPM returns execution results in the normalized `qhw-data` format. libquantum places the returned `qhw-data` object on the selected completion queue without interpreting its contents. The application owns the object after consuming the completion and may use `qhw-data` parsing APIs to access specific result data.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-rslt-001"></a>`LQ-RSLT-001` | libquantum shall place each `qhw-data` result returned by QPM on the task's selected completion queue without redefining or converting the result format. | Yes |
| <a id="lq-rslt-002"></a>`LQ-RSLT-002` | Ownership of a consumed `qhw-data` result shall transfer to the application, which is then responsible for retaining or releasing it. | Yes |
| <a id="lq-rslt-003"></a>`LQ-RSLT-003` | The completion containing a result shall identify the selected compiler/device route and the owning QPM task. | Yes |
| <a id="lq-rslt-004"></a>`LQ-RSLT-004` | Result transformations that change cost, sampling requirements, approximation, or statistical meaning shall require declared application configuration or policy. | No |
| <a id="lq-rslt-005"></a>`LQ-RSLT-005` | The library shall support explicitly configured application result-processing operations before publication of the associated logical completion, with processing failures reported through that completion. | No |
| <a id="lq-rslt-006"></a>`LQ-RSLT-006` | When circuit cutting is explicitly enabled, the library shall track generated fragment tasks and reconstruct the requested logical result before publishing successful completion. | No |

### Application Control

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-app-001"></a>`LQ-APP-001` | libquantum shall expose its public operations for direct use by application code while leaving algorithmic control and data flow with the application. | Yes |
| <a id="lq-app-002"></a>`LQ-APP-002` | libquantum shall allow applications to invoke the public execution and result-retrieval operations repeatedly without creating relationships between completed and subsequent tasks. | Yes |

### Diagnostics and Provenance

> **TODO:** Review these deferred requirements against the telemetry already maintained by QFw. Determine which information libquantum must correlate or forward without duplicating telemetry ownership or storage.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-obs-001"></a>`LQ-OBS-001` | libquantum shall report failures during selection, compilation, submission, and completion with the failure reason, available operation ID, and selected route or requested destination when known. | No |
| <a id="lq-obs-002"></a>`LQ-OBS-002` | Observable provenance shall distinguish the originating SDK when known, compiler and version when used, target description when available, QPM service/reservation, execution interface path, and any additional compilation stage that is known. Opaque submissions shall identify compilation by libquantum as bypassed. | No |
| <a id="lq-obs-003"></a>`LQ-OBS-003` | Where QFw telemetry is enabled, libquantum shall preserve task trace context and correlation rather than interrupting the existing job-to-provider trace. | No |
| <a id="lq-obs-004"></a>`LQ-OBS-004` | Provider timing shall be identified as optional and source-dependent. Missing timing or calibration values shall not be fabricated. | No |
| <a id="lq-obs-005"></a>`LQ-OBS-005` | Normal application diagnostics shall not expose provider secrets. | No |

### QFw Integration Boundaries

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-qfw-001"></a>`LQ-QFW-001` | Allocation and reservation creation shall remain owned by the launcher/resource manager and QFw admission infrastructure; libquantum shall consume the resulting context. | Yes |
| <a id="lq-qfw-002"></a>`LQ-QFW-002` | Execution shall use the managed QPM execution API and preserve admission, scheduler, credential, and accounting semantics. | Yes |
| <a id="lq-qfw-003"></a>`LQ-QFW-003` | libquantum shall discover services and obtain device metadata through public directory and telemetry bindings, and submit execution through the managed QPM API. | Yes |
| <a id="lq-qfw-004"></a>`LQ-QFW-004` | Compiler choice and the QRMI/QDMI execution-library choice are independent settings. Status, results, metadata, and cancellation shall use the path that owns the submitted task. | Yes |
| <a id="lq-qfw-005"></a>`LQ-QFW-005` | Existing QFw payload negotiation, qhw data meanings, lifecycle bindings, and event mechanisms shall be reused where they express the required behavior. | Yes |
| <a id="lq-qfw-006"></a>`LQ-QFW-006` | Changes needed to expose reusable QFw helpers shall be identified explicitly; constructing an SDK execution backend shall not be a prerequisite for using libquantum. | Yes |

### Native Interfaces and Compiler Extensibility

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-ext-001"></a>`LQ-EXT-001` | libquantum shall provide a native C library and public C interface with Python wrappers and a communication path suitable for non-Python applications. | No |
| <a id="lq-ext-002"></a>`LQ-EXT-002` | The observable graph, program, device, task, and result semantics shall remain expressible across the native language boundary. SDK Python objects shall not define the only durable interchange contract. | No |
| <a id="lq-ext-003"></a>`LQ-EXT-003` | Native compiler integrations, including Qiskit's C API, shall accept only input features and runtime combinations supported by the selected native compiler interface and shall report unsupported combinations. | No |
| <a id="lq-ext-004"></a>`LQ-EXT-004` | libquantum shall discover passes supported by a compiler integration and report their identities, configuration options, and applicability. | No |
| <a id="lq-ext-005"></a>`LQ-EXT-005` | libquantum shall accept selections and compositions of supported compiler passes and obtain validation of pass compatibility and ordering from the compiler integration before compilation. | No |
| <a id="lq-ext-006"></a>`LQ-EXT-006` | libquantum shall support additional compiler plugins through a documented public extension contract. | No |

The SC26 Python implementation and its existing QFw runtime do not establish the native C interface.

## Constraints

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-con-001"></a>`LQ-CON-001` | Resource discovery shall operate within an existing allocation and authorization context. Service reachability shall not be interpreted as allocation ownership. | Yes |
| <a id="lq-con-002"></a>`LQ-CON-002` | A supported SDK/compiler/device route shall preserve program semantics and produce a payload accepted by the actual service. | Yes |
| <a id="lq-con-003"></a>`LQ-CON-003` | SDK independence shall provide execution through the common libquantum/QRI path without requiring removal of compiler support objects or vendor SDK usage inside QFw provider implementations. | Yes |
| <a id="lq-con-004"></a>`LQ-CON-004` | The general program model shall accommodate representations beyond gate circuits without claiming arbitrary conversion among gate, analog, hybrid, or compiled-runtime programs. | No |
| <a id="lq-con-005"></a>`LQ-CON-005` | libquantum shall keep graph ordering separate from application-owned data flow and algorithmic logic. | No |
| <a id="lq-con-006"></a>`LQ-CON-006` | Library-owned state shall remain distinguishable from externally owned allocation, reservation, and service state. | Yes |
| <a id="lq-con-007"></a>`LQ-CON-007` | Unsupported combinations shall fail clearly. Optional compiler/provider features shall not be presented as universally available. | Yes |

## Non-Goals

The following scope exclusions explain the boundaries of the identified requirements. They are not a deferred feature backlog.

- Replacing Qiskit, PennyLane, MQT, Catalyst, or another SDK's program construction and compiler internals.
- Reimplementing QFw admission, qhw scheduling, provider credential handling, device drivers, or DEFw transport.
- Requiring an SDK execution backend or device plugin for every application SDK.
- Guaranteeing that every SDK works with every compiler and every quantum device.
- Defining a new workflow language, automatically inferring application algorithms, or performing implicit result-to-parameter binding.
- Promising distributed entanglement or execution of one coherent quantum circuit across independent QPUs.
- Providing real-time QEC control, Backline placement, or transport/memory orchestration inside a quantum resource.

## SC26 Demonstration

The following categories define the SC26 implementation constraints, deployment obligations, and demonstration coverage. Their requirement IDs are part of the same overall requirements set. Requirements in the general categories with SC26 = Yes also apply; the demonstration is not limited to the tables in this section.

### Implementation and Environment

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-env-001"></a>`LQ-ENV-001` | The SC26 libquantum implementation shall use Python and the supported Python interfaces of its compiler integrations. | Yes |
| <a id="lq-env-002"></a>`LQ-ENV-002` | The SC26 libquantum implementation shall integrate with the activated QFw environment and its normal application launcher, reusing the deployed Python directory, telemetry, execution, and event APIs. | Yes |
| <a id="lq-env-003"></a>`LQ-ENV-003` | SC26 shall consume existing allocation/reservation context without introducing a second service plane or a new reservation protocol. | Yes |
| <a id="lq-env-004"></a>`LQ-ENV-004` | SC26 shall operate without depending on native application support or completion of DEFw2; the existing `defw2-prototype` work shall remain a separate development line. | Yes |
| <a id="lq-env-005"></a>`LQ-ENV-005` | The demo installation shall record the QFw commit, required PR commits, runtime and dependency versions, compiler versions, input formats, device/service identities, and necessary site configuration. | Yes |
| <a id="lq-env-006"></a>`LQ-ENV-006` | The Qiskit/MQT/PennyLane/Catalyst dependency set shall be qualified as an installable integrated environment; a successful import or isolated compiler test shall not establish deployment readiness. | Yes |
| <a id="lq-env-007"></a>`LQ-ENV-007` | SC26 direct program execution shall reuse supported asynchronous QFw behavior, and blocking libquantum waits shall observe the same submitted tasks. | Yes |

### Devices and Route Eligibility

The device and compiler route demonstration uses recognized gate programs; the application chooses among eligible routes. Opaque inputs use their configured backend as described in `LQ-PRG-002`.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-demo-001"></a>`LQ-DEMO-001` | SC26 shall target the allocated IBM machine described by the operator as 156 qubits and the IQM machine described as 20 qubits. Actual usable capacity and identity shall be discovered rather than assumed from those nominal sizes. | Yes |
| <a id="lq-demo-002"></a>`LQ-DEMO-002` | Both devices shall be discoverable in the same application's allocation context when SC26 compares or selects between them. | Yes |
| <a id="lq-demo-003"></a>`LQ-DEMO-003` | Device selection shall distinguish multiple services whose directory provider is `shim`. | Yes |
| <a id="lq-demo-004"></a>`LQ-DEMO-004` | SC26 shall demonstrate route queries by submitted representation/version, strict compiler/device constraints, deterministic route ordering, and application choice of an eligible route. | Yes |
| <a id="lq-demo-005"></a>`LQ-DEMO-005` | SC26 shall demonstrate that a small program has an eligible IQM route and that a program exceeding IQM's usable capacity excludes IQM while retaining an eligible IBM route; the application chooses the route to open. | Yes |
| <a id="lq-demo-006"></a>`LQ-DEMO-006` | Device readiness, gate and feature support, and compiler/output compatibility shall remain eligibility checks when qubit capacity is used as an eligibility constraint. | Yes |
| <a id="lq-demo-007"></a>`LQ-DEMO-007` | An unavailable target or insufficient compiler target metadata shall produce a clear readiness/selection error. A generic fallback target shall not be presented as a measured hardware target. | Yes |

### Interoperability Scope

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-demoint-001"></a>`LQ-DEMOINT-001` | SC26 shall demonstrate two quantum chemistry applications through the common libquantum submission path: one using the variational quantum eigensolver (VQE) approach and one using the sample-based quantum diagonalization (SQD) approach. | Yes |
| <a id="lq-demoint-002"></a>`LQ-DEMOINT-002` | SC26 shall expose Qiskit, MQT Core, and Catalyst compiler integrations with visible availability and compatibility, and shall qualify at least one runnable QRI route per compiler. A complete SDK/compiler/device cross-product is not required. | Yes |
| <a id="lq-demoint-003"></a>`LQ-DEMOINT-003` | SC26 shall demonstrate managed QPM execution on both allocated physical IBM and IQM devices, with applications consuming results according to the program's compiler/runtime contract. | Yes |
| <a id="lq-demoint-004"></a>`LQ-DEMOINT-004` | SC26 shall demonstrate at least one SDK-origin program compiled by another toolchain and executed through QRI without the application instantiating an SDK execution backend. | Yes |
| <a id="lq-demoint-005"></a>`LQ-DEMOINT-005` | SC26 shall demonstrate separate application route choice, compilation, and QRI submission calls, and operation/result observation through public APIs and completion queues. | Yes |
| <a id="lq-demoint-006"></a>`LQ-DEMOINT-006` | SC26 shall demonstrate multiple local completion queues and shared-queue operation correlation through QRI `cid`. | Yes |

A runnable Catalyst-to-QRI path remains a pending integration gate for `LQ-DEMOINT-002`. The existing Backline executor path bypasses QRI execution and therefore does not demonstrate that requirement. The reviewed upstream evidence is retained in the QFw Upstream Review.

### Programs and Compiled Artifacts

For recognized programs and compiler artifacts, libquantum selects payload formats according to the requirements below. Opaque blobs follow `LQ-PRG-002`; payload interpretation and acceptance belong to the receiving backend.

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-fmt-001"></a>`LQ-FMT-001` | SC26 shall declare the supported subset of ordinary gate programs and parameter binding, with input representations sufficient for its qualified Qiskit, MQT, and Catalyst routes. | Yes |
| <a id="lq-fmt-002"></a>`LQ-FMT-002` | libquantum shall query each allocated QPM through QRI during startup for its accepted program-input representations and exact versions using the versioned program-input-capability contract. | Yes |
| <a id="lq-fmt-003"></a>`LQ-FMT-003` | Payload selection shall use each QPM's actual declarations. The reviewed IBM shim requires QPY-based input and removes `openqasm2`; IQM input shall use QPY or OpenQASM 2 only where advertised. | Yes |
| <a id="lq-fmt-004"></a>`LQ-FMT-004` | OpenQASM 3, MLIR, or QIR compiler intermediates shall be submitted only through an explicitly supported QFw execution-format integration; compiler support alone shall not imply QPM acceptance. | Yes |
| <a id="lq-fmt-005"></a>`LQ-FMT-005` | libquantum shall forward application-supplied program data, parameter values, and shots to the owning compiler/runtime integration under its declared interface. | Yes |
| <a id="lq-fmt-006"></a>`LQ-FMT-006` | Required provider serialization may remain below libquantum, but any additional compiler stage or remapping shall be visible and shall preserve the selected compiler experiment's interpretation. | Yes |

Existing Pulser support illustrates per-resource format discovery. Pasqal execution remains outside the IBM/IQM SC26 scope.

### Required QFw Contracts

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-adapt-001"></a>`LQ-ADAPT-001` | libquantum shall join the active QFw runtime supplied by the application launcher, use the existing DEFw Python bridge, and preserve launcher ownership. | Yes |
| <a id="lq-adapt-002"></a>`LQ-ADAPT-002` | libquantum shall read `QFW_RESERVATIONS` as a JSON list of `[service_id, reservation_id]` pairs, retain every authorized pair, and normalize decimal-string reservation IDs consistently at service boundaries. | Yes |
| <a id="lq-adapt-003"></a>`LQ-ADAPT-003` | libquantum shall resolve reserved services/devices through directory records and managed execution/telemetry bindings while retaining runtime and generation identity. | Yes |
| <a id="lq-adapt-004"></a>`LQ-ADAPT-004` | libquantum shall obtain permitted public target metadata through the applicable QFw telemetry calls, including `get_backend_info`, `get_device_info`, `get_coupling_graph`, `get_calibration_snapshot`, and `capability_map`, and shall qualify availability in the demo deployment. | Yes |
| <a id="lq-adapt-005"></a>`LQ-ADAPT-005` | libquantum shall reuse supported QFw telemetry-to-target conversions without requiring `QFwBackend` construction, and shall reject or clearly label generic fallback metadata. | Yes |
| <a id="lq-adapt-006"></a>`LQ-ADAPT-006` | libquantum shall preserve reservation/token context, QRI `cid` operation IDs, and managed lifecycle outcomes when using `api_qpm_execution.async_run`, `task_status`, and `cancel_task`. | Yes |
| <a id="lq-adapt-007"></a>`LQ-ADAPT-007` | libquantum shall retain service, reservation, route, and QRI `cid` identity while delivering completions to the queue selected for that operation obtained through QFw completion queues or event notifications, and shall unregister owned listeners. | Yes |
| <a id="lq-adapt-008"></a>`LQ-ADAPT-008` | libquantum shall reuse `util.circuit_payload` and QPM directory properties for supported serialization and version negotiation without introducing a parallel wire envelope. | Yes |
| <a id="lq-adapt-009"></a>`LQ-ADAPT-009` | Provider sessions, credentials, throttling, and per-device scheduling shall remain with QFw and its QRMI/QDMI provider paths. | Yes |
| <a id="lq-adapt-010"></a>`LQ-ADAPT-010` | libquantum shall preserve trace context through existing `qfw.*` instrumentation and shall identify additional libquantum compilation time separately from optional provider timing. | Yes |

These requirements name integration surfaces for qualification. They do not imply identical capability coverage or telemetry method signatures across every QPM.

### Integration Readiness Gates

| Requirement ID | Requirement description | SC26 |
| --- | --- | --- |
| <a id="lq-qual-001"></a>`LQ-QUAL-001` | SC26 qualification shall freeze the QFw, cluster, and compiler environment and identify included open PR commits, distinguishing main, development branches, and deployed builds. | Yes |
| <a id="lq-qual-002"></a>`LQ-QUAL-002` | SC26 qualification shall prove that one allocation exposes both devices with the correct service/reservation pairs and sufficient workload allowance. | Yes |
| <a id="lq-qual-003"></a>`LQ-QUAL-003` | SC26 qualification shall obtain real compiler targets for both devices using the deployed introspection configuration and permitted credentials, without giving application code provider secrets. | Yes |
| <a id="lq-qual-004"></a>`LQ-QUAL-004` | SC26 qualification shall exercise each selected compiler through its Python interface and convert its returned object into a payload accepted by the selected QPM. | Yes |
| <a id="lq-qual-005"></a>`LQ-QUAL-005` | SC26 qualification shall verify that application parameter values and the selected compiler result reach the owning integrations and that those integrations preserve the program's execution and result contracts, while recording QFw-side recompilation. | Yes |
| <a id="lq-qual-006"></a>`LQ-QUAL-006` | SC26 qualification shall prove at least one Catalyst route through QRI; Backline reservation/executor support shall be assessed separately. | Yes |
| <a id="lq-qual-007"></a>`LQ-QUAL-007` | SC26 qualification shall exercise independent asynchronous submissions under deployed QFw limits, task failure, cancellation, nonterminal timeouts, multiple completion queues, and listener cleanup through direct public APIs. | Yes |
| <a id="lq-qual-008"></a>`LQ-QUAL-008` | SC26 qualification shall identify physical IBM/IQM results separately from mock or simulator results; mock success shall not replace hardware interoperability evidence. | Yes |

### Requirements Outside the SC26 Milestone

All rows marked **No** remain in the overall requirements set. The principal groups are:

- Direct submission of stored compiled artifacts: `LQ-PRG-009`.
- Automatic route-selection policies and their configuration or explanations: `LQ-SEL-004`, `LQ-SEL-005`, and `LQ-POL-001` through `LQ-POL-008`.
- Checks after compilation and automatic-selection stability after remote acceptance: `LQ-SEL-006` and `LQ-SEL-007`.
- Optional graph construction, quantum-program metadata, and later classical task nodes: `LQ-GRF-001` through `LQ-GRF-004`.
- Optional sequential/concurrent graph execution through the shared public APIs: `LQ-EXE-001` through `LQ-EXE-005`; graph ordering and application data flow remain separate under `LQ-CON-005`.
- Result transformations, application-defined result processing, and explicitly enabled circuit-cutting reconstruction: `LQ-RSLT-004` through `LQ-RSLT-006`.
- Diagnostics and provenance: `LQ-OBS-001` through `LQ-OBS-005`.
- Representation of program families beyond the initial gate-program subset: `LQ-CON-004`.
- Native C interfaces, compiler pass discovery/composition, and public compiler-plugin extension support: `LQ-EXT-001` through `LQ-EXT-006`.

General workflow-language constructs, implicit result-to-input binding, automatic algorithm ownership, and distributed entanglement remain scope exclusions. Adding Pasqal, Braket, or resource-side Backline/QEC execution to SC26 is not implied by this roadmap.

## Acceptance Criteria

Acceptance criteria reference the requirement IDs defined in the category tables. They describe expected qualification evidence and do not create a second set of requirements. These cases have not been executed as part of document authoring.

### General Acceptance

| Requirement IDs | Acceptance evidence | SC26 |
| --- | --- | --- |
| `LQ-INT-001`, `LQ-INT-002`, `LQ-INT-003` | Direct calls to route selection, compilation, SDK-circuit execution, and task/completion APIs provide the same supported behavior across SDK/compiler/device combinations; compilation-only calls return artifacts for inspection without device submission. | Yes |
| `LQ-CMP-001`, `LQ-CMP-002`, `LQ-CMP-003`, `LQ-CMP-004` | Compiler integrations register callbacks; libquantum queries input/output formats and versions at startup, uses valid responses for mapping and selection, and invokes compatibility and compilation through the common callback contract. | Yes |
| `LQ-COMP-001`, `LQ-COMP-002`, `LQ-COMP-003`, `LQ-COMP-004` | Before the first program submission, reported compiler outputs and QRI device inputs enumerate all matching format/version routes. Native and internally converted inputs index the same compiler/output/device routes; classification is exposed as route metadata and does not change structural route identities. | Yes |
| `LQ-COMP-007` | Explicit query API calls return structured route records or compatibility details, including reasons for missing or rejected routes, that callers can consume programmatically. | Yes |
| `LQ-PRG-004`, `LQ-RSLT-001`, `LQ-RSLT-002`, `LQ-RSLT-003`, `LQ-CQ-006` | Program objects and supplied inputs reach the owning integration. QPM results reach the selected completion queue as `qhw-data` without conversion, identify the execution route and QPM task, and transfer to application ownership when consumed. | Yes |
| `LQ-PRG-002`, `LQ-TASK-002`, `LQ-QFW-002`, `LQ-CQ-001`, `LQ-CQ-004` | An unidentified blob reaches the configured destination unchanged, with no libquantum parsing, conversion, or compiler invocation; shared task and completion APIs deliver its outcome to the selected queue. | Yes |
| `LQ-COMP-004`, `LQ-DEV-004`, `LQ-TASK-002`, `LQ-CQ-001`, `LQ-FMT-005` | Two independent program tasks share a compiler/output/device route while supplying different execution options and completion queues; each integration call receives that task's inputs and each completion reaches its assigned queue. | Yes |
| `LQ-SEL-001`, `LQ-SEL-002`, `LQ-SEL-003`, `LQ-SEL-008` | Public route queries apply explicit compiler/device constraints, return every eligible route in deterministic order, and leave the final choice to the application. | Yes |
| `LQ-COMP-006`, `LQ-SEL-003` | A 50-qubit program whose only compatible route ends at a 20-qubit device fails selection before device submission, even when a larger device exists without a compatible route. | Yes |
| `LQ-POL-001`, `LQ-POL-002`, `LQ-POL-003` | Compiler capability metadata distinguishes native and internally converted inputs, while capacity and compatibility determine eligibility; automatic ranking remains future scope. | No |
| `LQ-TASK-001`, `LQ-TASK-002`, `LQ-PRG-001`, `LQ-CMP-003`, `LQ-COMP-008`, `LQ-CQ-002` | A compilation-only call returns compiler output without QPM submission. A separate `run()` call accepts that compiler result, submits it through the selected route, and returns QRI `cid` as the operation ID; compilation failures are returned by `compile()` before submission. | Yes |
| `LQ-TASK-002`, `LQ-TASK-003`, `LQ-TASK-005`, `LQ-CQ-002`, `LQ-CQ-003`, `LQ-CQ-004`, `LQ-CQ-006` | Multiple operations remain outstanding where permitted. Status and cancellation use the owning QPM and tracked QRI `cid`; a caller-side wait timeout does not change QPM status or publish a completion. | Yes |
| `LQ-FIN-001`, `LQ-FIN-003`, `LQ-FIN-004` | Finalization reports unfinished work, unregisters owned listeners, and is safe to repeat without releasing external allocations or services. | Yes |
| `LQ-PRG-009` | A compatible stored compiled artifact can be submitted through an explicit artifact input contract using its selected route and runtime inputs, with task status and results observable through the public APIs. | No |
| `LQ-SEL-004`, `LQ-SEL-005` | The application policy configuration interface changes enabled policies and preference order; explicit selection inspection returns the effective route, supplied options, and decision reasons. | No |
| `LQ-SEL-006`, `LQ-SEL-007` | Checks after compilation account for target and capacity changes, and remote acceptance or an uncertain submission outcome preserves the selected hardware, compiler, and execution path. | No |
| `LQ-POL-004`, `LQ-POL-005`, `LQ-POL-006` | Supplied trial artifacts or reported quality and timing estimates produce the expected eligible selections under each configured ranking metric. | No |
| `LQ-POL-007` | Related tasks prefer the previously selected compiler/device pair while that pair remains compatible. | No |
| `LQ-POL-008` | Independent nodes are distributed among compatible devices while dependency order and QFw admission and scheduling semantics are preserved. | No |
| `LQ-GRF-001`, `LQ-GRF-002`, `LQ-GRF-003`, `LQ-EXE-001`, `LQ-EXE-002` | Applications create sequential or concurrent quantum-program graphs with routing metadata; executor calls use the same public APIs and metadata as direct application calls. | No |
| `LQ-EXE-003`, `LQ-EXE-004`, `LQ-EXE-005`, `LQ-CON-005` | Sequential programs wait for prior success; concurrent programs can remain outstanding independently. Failures skip later sequential nodes, independent concurrent work can finish, and graph status reports node outcomes while application code owns data flow. | No |
| `LQ-GRF-004` | A later graph extension accepts explicitly supplied classical task nodes through its node API. | No |
| `LQ-RSLT-004`, `LQ-RSLT-005`, `LQ-RSLT-006` | Configured result transformations, post-processing, and circuit-cutting reconstruction produce the requested logical completion or a processing error. | No |
| `LQ-EXT-001`, `LQ-EXT-002`, `LQ-EXT-003` | A supported non-Python application uses the native interface while preserving the defined program, task, and result semantics. | No |
| `LQ-EXT-004`, `LQ-EXT-005`, `LQ-EXT-006` | Compiler passes are discoverable, additional compiler plugins can register through the public extension contract, supported configurations are accepted, and incompatible pass composition fails clearly. | No |

### SC26 Demonstration Acceptance

| Requirement IDs | Acceptance evidence | SC26 |
| --- | --- | --- |
| `LQ-RES-001`, `LQ-RES-002`, `LQ-DEMO-001`, `LQ-DEMO-002`, `LQ-ADAPT-002` | libquantum discovers the allocated IBM and IQM devices through the QRI, mirrors their reported capacity, metadata, and reservation information, and refreshes that mirror from QRI updates. | Yes |
| `LQ-INT-001`, `LQ-DEMOINT-001`, `LQ-DEMOINT-004` | The VQE and SQD quantum chemistry applications use the common libquantum submission path, with at least one verified cross-toolchain route and no SDK backend submission. | Yes |
| `LQ-DEMOINT-002`, `LQ-COMP-008` | The same supported logical program compiles through Qiskit and MQT with recorded target/version/options and successful QRI execution. | Yes |
| `LQ-DEMOINT-002`, `LQ-QUAL-006` | A Catalyst-compiled SDK-origin program produces a verified result through QRI; its conversion/runtime bridge and any subsequent compiler stage are recorded. | Yes |
| `LQ-DEMOINT-003`, `LQ-DEMO-005` | Supported workloads execute on both physical devices; route queries show the expected IQM/IBM eligibility from capacity and compatibility, and the application chooses each route. | Yes |
| `LQ-DEMO-004`, `LQ-SEL-001`, `LQ-SEL-003`, `LQ-SEL-008` | The demo enumerates routes, applies explicit constraints and eligibility checks, returns them in deterministic order, and lets the application open the chosen route. | Yes |
| `LQ-DEMOINT-005`, `LQ-INT-002`, `LQ-TASK-001`, `LQ-TASK-002`, `LQ-APP-002` | Application code independently queries and opens a route, compiles the SDK-origin program, submits the compiler result with `run()`, and observes operation status and completions through public APIs. | Yes |
| `LQ-DEMOINT-006`, `LQ-TASK-005`, `LQ-CQ-003`, `LQ-CQ-004` | Interleaved operation results reach their selected queues and carry the QRI `cid` returned as `op_id`. | Yes |
| `LQ-COMP-005`, `LQ-TASK-002`, `LQ-CQ-002`, `LQ-CQ-006` | Unsupported recognized compilation routes fail in route query or compilation before `run()`; QPM execution failures and timeouts produce terminal completions, and a caller-side wait timeout produces no completion or status change. | Yes |
| `LQ-TASK-004`, `LQ-FIN-004`, `LQ-CQ-007` | Cancellation and repeated finalization report remote outcomes accurately and leave no owned event registration behind. | Yes |
| `LQ-ENV-005`, `LQ-ENV-006`, `LQ-QUAL-001`, `LQ-QUAL-008` | A fresh installation reproduces the qualified routes with recorded revisions, dependency policy, and hardware-versus-mock provenance. | Yes |

Hardware outcomes need not be bit-for-bit identical across devices or compilers. Applications may use `qhw-data` parsing APIs to assess returned results against the program's result contract and appropriate statistical expectations.

## QFw Upstream Review

**Informative integration evidence, reviewed 2026-10-24.**

The upstream default branch was [`main` at `eacec30bee675e9c468099b8faa9330898a1e748`](https://github.com/openQSE/QFw/tree/eacec30bee675e9c468099b8faa9330898a1e748), whose latest merge was PR 173. The review covered the six open PRs, the 60 most recently updated merged PR results, relevant PR discussions, and the source files named below. This is not a claim of exhaustive history review or live hardware validation.

The separate `defw2-prototype` branch was at `b6629de89981e6e963ddc43dd8df2969ec78ba99`; its PR 146 merged into that development branch, not main. Its native/runtime work is not an SC26 prerequisite.

Findings below distinguish observed implementation, author-reported validation, and requirements derived for libquantum. PR test reports remain upstream reports; they were not rerun during requirements authoring.

### Merged Infrastructure and Implications

| Evidence at the reviewed main | Status and finding | Implication for libquantum |
| --- | --- | --- |
| [QPM API inventory](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/docs/qpmd-supported-apis.md) and [execution binding](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/service-apis/api_qpm_execution/api_qpm_execution.py) | Public execution, telemetry, admission, scheduler, and privileged-control APIs are separated. Execution includes cancellation, completion queues, and listener unregistration. | Bind execution and telemetry; leave reservation policy, provider scheduling, and service administration with QFw. |
| [Reservation-set parser](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/backends/qfw_qiskit/reservation_set.py) and [PR 145](https://github.com/openQSE/QFw/pull/145) | Allocation context supports multiple service/reservation pairs. Selecting by provider alone was ambiguous when several devices used `shim`. | Enumerate pairs and select by service/device identity; do not collapse all shim services into one device. |
| [PR 163](https://github.com/openQSE/QFw/pull/163) and [target helper](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/backends/qfw_qiskit/qfw_target.py) | Compiler targets can be built from public device/coupling/calibration telemetry, including IBM-specific descriptions and qhw records. Generic fallback remains possible. | Reuse target conversion outside backend submission and make target completeness/source visible. |
| [PRs 126](https://github.com/openQSE/QFw/pull/126) and [127](https://github.com/openQSE/QFw/pull/127) | IQM native conversion can compile abstract gates from architecture information on QRMI and QDMI paths; already native serialization preserves intended placement. | Record which stages compile, preserve mappings, and qualify the selected compiler experiment end to end. |
| [IBM transcoder](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/services/util/ibm_transcode.py) | QFw can compile abstract input to the IBM target and skips compilation when it recognizes already supported ISA operations. It also packages parameter bindings. | Distinguish selected compiler output from QFw fallback compilation and preserve binding/layout semantics. |
| [Payload contract](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/services/util/circuit_payload.py) and [shim advertisement](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/services/svc_lib_qpm/svc_qpm.py) | Formats include OpenQASM 2, QPY, compressed QPY, and Pulser. IBM removes OpenQASM 2 from its declaration. | Negotiate per resource; do not assume OpenQASM 3 or QIR is a directly accepted QPM payload. |
| [PR 147](https://github.com/openQSE/QFw/pull/147), [PR 156](https://github.com/openQSE/QFw/pull/156), and [QDMI pins](https://github.com/openQSE/QFw/blob/eacec30bee675e9c468099b8faa9330898a1e748/setup/qdmi-requirements.txt) | Released QDMI integrations are reused; reviewed pins include MQT Core 4.1.1, IQM QDMI 1.5.0, IBM QDMI 1.0.0, and Braket QDMI 1.3.1. | Use existing device integration and qualify the actual compiler environment; installed QDMI access is not evidence of compiler compatibility. |
| [PR 130](https://github.com/openQSE/QFw/pull/130) and [PR 131](https://github.com/openQSE/QFw/pull/131) | Reservation-owned sessions are evicted and task metadata follows the execution library that ran the task. | Keep service/reservation/task/path identity together and avoid cross-session handle lookup. |
| [PRs 100](https://github.com/openQSE/QFw/pull/100) and [103](https://github.com/openQSE/QFw/pull/103) | Dispatch races and leaked task capacity were addressed. PR 100 also reported limits around raised dispatch depth and mixed synchronous/asynchronous use. | Preserve managed submission semantics; qualify concurrency against the frozen build rather than assuming arbitrary parallel device execution. |
| [PR 109](https://github.com/openQSE/QFw/pull/109), [PR 155](https://github.com/openQSE/QFw/pull/155), and [issue 121](https://github.com/openQSE/QFw/issues/121) | Reservation ID normalization and listener teardown are implemented. PR 155 explicitly leaves duplicate delivery to multiple live listeners sharing a reservation unresolved. | Normalize IDs, bind each operation to its selected queue, and unregister listeners during cleanup. |
| [PR 115](https://github.com/openQSE/QFw/pull/115) and [PR 173](https://github.com/openQSE/QFw/pull/173) | Job/provider tracing is integrated; optional IQM vendor phase durations were added. PR 173 reports no live q20 validation for that addition and no equivalent QDMI timeline. | Preserve traces, distinguish client compilation from provider phases, and report unavailable timing honestly. |
| [PR 158](https://github.com/openQSE/QFw/pull/158) | Pasqal accepts Pulser programs through a format-specific route. | Preserve a format-aware general model while keeping Pasqal outside the IBM/IQM demo scope. |
| [PR 159](https://github.com/openQSE/QFw/pull/159) | Backline placement launches a `catalyst-executor`; the new QRC refuses QRI execution calls. Its prototype is reservation/resource-side support. | Do not count this as a generic Catalyst-to-QRI compiler integration or as proof of the libquantum execution goal. |

### Open Pull Requests Affecting SC26

All entries were **open and unmerged** at the review snapshot. Including their functionality in the demo requires an explicitly recorded merged build, pinned PR revision, or equivalent supported integration.

| Pull request and reviewed head | Proposed behavior | SC26 consequence |
| --- | --- | --- |
| [PR 171](https://github.com/openQSE/QFw/pull/171), `3d1f714eea8a0a5f10a69eaae178b7a5b6f2e285` | Adds `QFwBackend.compile` with Qiskit/MQT choices and an MQT helper. Declares a dependency on PR 170. | Reuse the compiler work through a common integration without requiring backend construction. Verify compiler result serialization, options, layouts, and execution. |
| [PR 170](https://github.com/openQSE/QFw/pull/170), `62a0b078fea0b7c41291cf8b8af64df40137632f` | Moves Qiskit from main's 2.2.3 pin to 2.5.2 and overrides PennyLane-Qiskit 0.45.0's declared cap. | An installable, tested dependency policy is a release gate, not a presumed solved dependency. |
| [PR 168](https://github.com/openQSE/QFw/pull/168), `49d896306722d05541ef51e22df7b98faf418591` | Forwards `param_values` and `compilation_options` through backend/job submission and adds IBM examples. | The direct libquantum QPM adapter must preserve equivalent fields regardless of backend adoption. |
| [PR 169](https://github.com/openQSE/QFw/pull/169), `fdad55c48d1c82d99c1c685163c236cd3127dfdc` | Adds a Garnet mock shim with metadata useful for target-aware MQT examples; fake IQM's target data is insufficient for that purpose. | Qualify with representative target metadata; a canned-count fake cannot establish compiler or hardware correctness. |
| [PR 172](https://github.com/openQSE/QFw/pull/172), `f459f6f0e0fc12db93e017007042cd81c3baab15` | Proposes QRMI HTTP 429 handling, polling backoff, and a per-device concurrent-job limit. Its description records unresolved provider/introspection limitations. | Concurrent task submissions must respect deployed QFw limits. Throttling and retries belong to QFw; do not assume this open patch covers every provider operation or timeout. |
| [PR 52](https://github.com/openQSE/QFw/pull/52), `3686efbaab567731a1cfa3e5938d1b86be759632` | Proposes Docker dependency/build packaging. | Packaging work is tracked separately; it is not evidence that the demonstration's integrated dependency set is already installed. |

[PR 170's review discussion](https://github.com/openQSE/QFw/pull/170#issuecomment-6374803438) identifies that the ordinary pip installation cannot apply the proposed override, and that replacing a requirement may omit QASM import extras. The frozen demo recipe must resolve both issues if that route is used.

[PR 171's discussion](https://github.com/openQSE/QFw/pull/171#issuecomment-6383050338) points to newer upstream MQT batch support on main. A development feature must not be assumed present in QFw's pinned MQT release.

### Unresolved Integration Gaps

- **Target authorization and completeness.** Main's `get_backend_info` accepts a reservation ID, but the other telemetry methods do not expose the same reservation argument, and the existing telemetry convenience wrapper primarily supplies library/token context. PR 163 reports a real target falling back to generic metadata when service-side credentials or capability configuration were insufficient. Every selected telemetry path must be qualified under the demo's actual user and service setup.
- **Site configuration is part of compiler readiness.** PR 163 reports IBM and other site entries with execution-only capability maps. Merging target-builder code alone does not expose the introspection calls a compiler needs.
- **Backend-independent reuse.** Current resolver, target, and compiler helpers reside partly under `backends/qfw_qiskit`. Their location does not require backend execution, but reuse may need an upstream public helper boundary. Requirements must not couple libquantum to private driver internals.
- **Compiler artifact handoff.** PR 171's helper returns the MQT compiler result directly. The accepted representation, parameter/layout preservation, and serializer path must be verified for the chosen version and device.
- **Catalyst.** The reviewed main and open PR set establish Backline resource placement, not a generic Catalyst compiler artifact route through QRI to IBM/IQM. Input conversion, runtime/lowering, and accepted QPM output remain explicit integration work.
- **Completion routing.** Listener cleanup is merged, but broadcast delivery to multiple live listeners remains an upstream limitation. The selected queue registration and QRI `cid` must identify each libquantum operation without introducing a central redistribution layer.
- **Deployment evidence.** PR merge status, upstream author-reported tests, container contents, and live device readiness are different facts. Demo acceptance requires evidence from the frozen deployed stack.

## Open Decisions

These decisions belong in detailed design or demo qualification. They do not weaken the required execution goal.

- Select the exact supported Qiskit and PennyLane input subset and the conversions provided internally by each compiler integration.
- Define the Catalyst-to-QRI route and determine whether its runtime integration belongs in a compiler adapter or an upstream resource implementation.
- Freeze an installable version set, including QFw/cluster PR revisions, Qiskit, PennyLane, MQT Core, and Catalyst. Document any temporary dependency override.
- Define the future graph/executor APIs and later classical-node support around the same public program and task APIs; this work is outside SC26.
- Define common representation identifiers, format-version compatibility rules, and callback response fields for native and internally converted input capabilities used in startup route enumeration and selection.
- Keep the public lifecycle aligned with the detailed design: application route choice, separate compilation, `op_id = run(...)`, QPM-backed status/cancellation, and completion by the same QRI `cid`.
- Determine whether the demonstration needs optional application-visible progress. Application-defined result processing and circuit cutting remain future scope.

## Design Investigation Notes

These notes identify design work that must be resolved before the affected requirements become a detailed design. They are informative and do not add normative requirements.

### Program-format discovery

Current QFw QPM directory advertisements already expose a preferred `circuit_formats` list, and QPY-capable services expose `qpy_version`. This provides limited format discovery for OpenQASM 2, QPY, and compressed QPY. It is not yet the general per-format and per-version compatibility contract assumed by libquantum. In particular, the generic QPM query helper supplies a default OpenQASM 2 advertisement, while the current Backline QRC rejects all program-execution calls. The advertised format therefore does not always prove that a usable execution route exists.

**TODO:** Define an authoritative QPM/QRI input-format advertisement in which every reported format and version is accepted by that execution path. Determine whether the existing QFw properties can be extended or require a new public query contract. libquantum shall consume this information rather than hard-code device formats.

### Compiler registration

The reviewed Qiskit, MQT Core, and Catalyst integrations expose compiler-specific operations, but they do not implement the common libquantum registration and callback contract described in this document. The libquantum compiler adapters and callbacks for identity, version, native and internally converted input formats, output formats, compatibility, and compilation are new integration code.

**TODO:** Confirm the minimum information available from each compiler and implement only the adapters needed for the SC26 routes. Do not treat the proposed callback contract as functionality already supplied by the compiler projects.

### Backline integration

The intended libquantum flow is to query QEC resource information through QPM/QRI, provide that information to the Catalyst compiler integration, and submit the compiled program through QRI to the selected Backline executor. QPM would then remain responsible for task status, completion, and result delivery.

The current Backline prototype implements only part of that flow:

| Desired step | Current implementation |
| --- | --- |
| Discover QEC resource capabilities | **Partial.** The Backline QPM advertises QEC codes, controller and coprocessor kinds, transports, and a placement schema version. |
| Obtain a concrete target placement | **Implemented.** A reservation can return controller device and wire information, decoder symbol and library, transport, and the address of a reservation-owned `catalyst-executor`. |
| Compile using the selected placement | **Partial.** The placement supports the existing client-driven Catalyst flow, but no libquantum Catalyst registration adapter consumes it through the proposed compiler callbacks. |
| Submit the compiled program through QRI | **Not implemented.** The Backline QRC rejects synchronous and asynchronous QRI execution calls. The client sends the Catalyst-compiled object directly to `catalyst-executor`. |
| Observe execution through QPM status, cancellation, completion queues, and `qhw-data` results | **Not implemented for Backline execution.** The Backline QRC execution, cancellation, completion, timing, and metadata methods are refusal or placeholder paths. |
| Execute against the intended quantum target | **Not implemented by the default prototype.** Its inventory loads Catalyst's `null.qubit` device plugin rather than an IBM, IQM, or other QRI-managed quantum device. |

The current implementation therefore demonstrates QFw admission, QEC resource matching, placement, and executor lifecycle. It does not yet demonstrate the desired end-to-end libquantum → Catalyst → QRI → Backline execution route. The design must determine the compiled-artifact contract accepted by QRI, how the QPM forwards that artifact to `catalyst-executor`, and how QPM task and result semantics cover that execution.

Evidence: [QFw circuit payload contract](https://github.com/openQSE/QFw/blob/main/services/util/circuit_payload.py), [QPM query helper](https://github.com/openQSE/QFw/blob/main/services/util/qpm/util_qpm.py), [Backline QPM](https://github.com/openQSE/QFw/blob/main/services/svc_backline_qpm/svc_qpm.py), [Backline matcher](https://github.com/openQSE/QFw/blob/main/services/svc_backline_qpm/matcher.py), [Backline QRC](https://github.com/openQSE/QFw/blob/main/services/svc_backline_qpm/svc_qrc.py), and [Backline inventory](https://github.com/openQSE/QFw/blob/main/services/svc_backline_qpm/inventory.yaml).
