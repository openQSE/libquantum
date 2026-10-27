---
type: markdown
---

# Quantum HPC Interfaces HLD

# 1\. Quantum Platform Manager (QPM)

The Quantum Platform Manager (QPM) API provides a unified abstraction layer for quantum hardware, enabling the software stack to remain hardware-agnostic by exposing hardware capabilities and features consistently. However, given the diverse modalities and varying hardware capabilities, a single generic API risks being overly broad, making it challenging to express hardware configurations and features effectively. To address this, the QPM API is divided into three categories. The first, Resource Management APIs, allows the resource manager to allocate resources for hybrid applications. The second, Runtime APIs, enables hybrid applications to execute quantum tasks during operation. The third category, still under investigation, includes hardware-specific APIs that focus on low-level primitives, such as gate and pulse level control, tailored to unique hardware features. These hardware-specific APIs are designed to map directly to hardware capabilities, while more complex logical operations are handled by higher-level layers, such as the Quantum Programming Interface (QPI).

\[Diagram omitted from text conversion; see source document.\]

## 1.1. Resource Management APIs

Resource Management APIs

API

Description

Prototype Status

qpm\_acquire()

Acquire the hardware device and optionally sets the hybrid job priority. Device keeps track of job IDs which are currently using the device.

Implemented in a simpler form in the prototype

qpm\_release()

Release the hardware device for this job.

Implemented in a simpler form in the prototype

qpm\_estimator()

Estimate how much capacity this hybrid job will require on the quantum computer, given a set of parameters; e.g., max number of circuits the application will execute, max size, etc

Under development

qpm\_status()

Request device status. Device status can be ready, error, calibrating, or busy.

Under development

## 1.2. Runtime and Hardware-specific APIs

Runtime APIs

API

Description

Prototype Status

qpm\_run\_qtask()

Request the execution of a quantum task. The API provides a datastructure which needs to be filled out by the caller, describing the quantum task. It also provides a way to specify a set of metadata, such as number of shots. The API queues the quantum task for execution and returns a unique qtask ID to be used to reference the task later on.

Implemented in the prototype. The qtask structure is implemented as a python dictionary that takes an openQASM string, number of shots, and other data related to simulators, since the prototype is implemented against simulators.

qpm\_abort\_qtask()

Abort a running quantum task provided its unique ID.

Implemented in the prototype.

qpm\_get\_qtask\_status()

Request the current status of the quantum task provided its unique ID. The status can be running, completed, or queued. It can include other metrics like how long it's been running, how long it's been queued and when it's expected to be scheduled for execution.

Implemented in a simpler form in the prototype. It returns running, completed, or queued status.

qpm\_get\_qtask\_result()

Request the result of a quantum task provided its unique ID. Beside the measurement bit strings, the result will include various metadata about the quantum task run, including error rates of the used gates, the length of time it stayed on the queue, and length of execution.

Implemented in a simpler form in the prototype. After the prototype framework simulates the circuit, it returns the simulator results as well as various statistics about the run; e.g., the queuing times, execution time, etc.

qpm\_get\_qtask\_list()

Request a list of current quantum tasks running or queued on the system.

Implemented in the prototype.

register\_event\_notification()

Request notifications to be pushed to the client instead of the client polling for events. Requester can specify the type of notification it requires.

  - **Quantum task completion:** Push the results of a quantum task when it completes
  - **Noise event:** Reports significant noise-related incidents, such as coherence time degradation or gate error spikes. For example, a sudden increase in T1 relaxation time for a superconducting qubit.
  - **Calibration event:** Signals when a calibration routine (e.g., tuning qubit frequencies, laser wavelengths) starts, completes, or fails. For example, a trapped-ion system recalibrates laser intensities.
  - **Hardware fault:** Indicates physical or operational failures, such as qubit decoherence, control electronics malfunctions, or trap instability.
  - **Environmental changes:** Reports changes in external conditions affecting performance, like temperature, magnetic fields, or vacuum pressure.
  - **Qubit State Anomaly:** Detects unexpected qubit behavior, such as unintended state flips or leakage to higher energy levels.
  - **Control System Error:** Signals issues in the control system, such as pulse generation failures or timing mismatches.
  - **Queue status change:** Updates the status of the task queue, such as task submission, prioritization, or cancellation.
  - **Software updates or configuration changes:** Signals updates to the quantum runtime, firmware, or control software, or changes in configuration settings.

Implemented in a simpler form in the prototype. Client can register for quantum task completion.

qpm\_get\_qubit\_connectivity\_map()

Request qubit connectivity map of the underlying hardware.

Under development.

qpm\_get\_gate\_error\_rate()

Request current gate error rates.

Under development.

qpm\_get\_decoherence\_time()

Request decoherence times.

Under development.

Hardware-Specific APIs

We are in the process of establishing a working group with vendors to identify the full set of hardware features that should be exposed to the software stack and to design APIs that represent them effectively.

# 2\. Quantum Programming Interface (QPI)

The Quantum Programming Interface (QPI) is designed to provide an application-friendly API that abstracts higher-level functionalities, eliminating the need for hardware vendors implementing the Quantum Platform Manager (QPM) to redundantly develop these constructs. By standardizing common high-level features, the QPI simplifies the interface between applications and diverse quantum hardware while bridging gaps between different hardware modalities. This approach aligns with recent advancements in quantum computing, such as the Quantum Hardware Abstraction Layer (QHAL), a multi-level interface developed collaboratively across multiple qubit technologies to optimize the quantum-classical interface for efficiency and portability. Although the QPM includes a third category of hardware-specific APIs, which may introduce feature discrepancies across modalities, the QPI's role is to ensure a consistent interface for applications. Unlike comprehensive quantum programming frameworks such as Qiskit or Pennylane, the QPI does not aim to offer a full quantum programming paradigm; instead, it provides a high-level set of constructs to facilitate uniform and straightforward integration with the underlying hardware. However, recognizing the importance of performance, the QPI, in conjunction with the QPM, allows for hardware-specific optimizations through its third category of APIs, which can be utilized when higher efficiency is required—thus balancing the trade-offs between abstraction and resource efficiency, as highlighted in recent research on breaking abstractions for resource-efficient quantum computing.

Furthermore, the QPI empowers applications to configure the quantum tool pipeline, enabling tailored optimization of quantum tasks. Through its tool management APIs, users can specify compiler passes, such as circuit cutting or gate reduction, and allocate these tasks across high-performance computing (HPC) nodes for efficient processing. The QPI configures the software stack based on available quantum resources, establishes connections to quantum hardware, and customizes final transpilation passes to suit specific backends. It abstracts the complexity of managing multiple resource backends, allowing seamless execution across diverse systems. For fine-grained control, the QPI introduces the q-stream construct, which groups quantum tasks (q-tasks) for sequential or parallel execution. Users can assign a q-stream to a specific quantum resource or let the QPI dynamically allocate resources based on availability, enhancing flexibility and efficiency. These functionalities, detailed in the sections below, position the QPI as a critical layer for optimizing hybrid quantum-classical workflows.

## 2.1. Object View

\[Diagram omitted from text conversion; see source document.\]

\[Diagram omitted from text conversion; see source document.\]

The QPI design abstracts the interface to one or more quantum resources through a set of logical constructs.

At the core is the **Mesh**—an object that can contain one or more quantum devices. Operations applied to the Mesh affect all devices it contains. These operations are managed by a **q-stream** object, which acts as a container for a sequence of operations that can be executed together. Operations within a q-stream can range from simple device queries to full quantum task execution.

Multiple q-streams can be bound to a single Mesh. Each q-stream represents a set of operations. Stream attributes can be set to run all operations on the stream in sequence or in parallel. A Mesh can also be configured to distribute operations across all constituent devices, either in an embarrassingly parallel form or through more complex entanglement procedures. For example, a quantum circuit may be executed in a distributed fashion across multiple devices within the same Mesh.

Each quantum resource is represented as a **device** object. A Mesh may contain one or more devices, and each device can also have one or more q-streams bound to it. In this case, q-streams behave similarly to those bound to a Mesh, but operations are restricted to the specific device.

An **event queue** object receives events from a device and is bound to it. There can be zero or more event queues associated with a device. If no event queues are bound, events from the device will not be captured. A single event queue receives all device events, but multiple queues can be created to handle different event types.

A **completion queue** is bound to a q-stream and receives notifications when operations in the q-stream complete. These notifications may not arrive in the same order as the operations were submitted if the q-stream is configured to run objects in parallel, otherwise the QPI ensures that notifications are ordered. A q-stream can have one or more completion queues, with each queue dedicated to a specific type of completion—for example, one for quantum task results and another for calibration outcomes.

Finally, the **tool pipeline** object manages tools that are applied to quantum tasks. A single tool pipeline can be bound to multiple q-streams, ensuring that the same set of tools is applied to all quantum tasks originating from those q-streams.

## 2.2. API

The following APIs are currently in the design stage.

Initialization APIs

API

Description

qpi\_init()

Initialize the QPI library. As part of the initialization the QPI library readies the available resources by reading all allocation environment variables.

qpi\_finalize()

Finalize the QPI library. Ensure all resources are cleaned up and freed.

Object Management

qpi\_getinfo()

Request information about devices available in the allocation. The device information block contain information like the qubit count, gate error rate, calibration cycle, etc.

qpi\_mesh()

Create a mesh object, given a set of attributes which define a super-set of all the device capabilities which can be added to the mesh.

qpi\_device()

Create a device object and add to the mesh. The device must take exactly one information block describing the device. If the device doesn't conform to the mesh attributes, the call will fail.

qpi\_qstream()

Create a stream object given a set of attributes that define the capabilities of the stream. E.g., parallel or sequential execution. 

qpi\_eq\_creat()

Create an event queue object given a set of attributes that define the queue. E.g., type of event the queue accepts.

qpi\_cq\_create()

Create a completion queue object given a set of attributes that define the queue. E.g., type of completion the queue accepts.

qpi\_tp\_create()

Create a tool pipeline object.

qpi\_bind()

Bind one object to another.

Tool Pipeline Control

qpi\_tp\_list()

Given a device ID, list all tools which can work with this device.

qpi\_tp\_prop()

Given a tool ID, return the properties of the tool as defined by the tool developer. E.g., tool category, node count it requires, and device type support.

qpi\_tp\_add()

Add a tool to the specified tool pipeline.

qpi\_tp\_remove()

Remove a tool from the specified tool pipeline.

Event Management

qpi\_eq\_read()

Read events from the event queue.

qpi\_cq\_read()

Read a completion notification from the completion queue.

qpi\_peek()

Peek completions without removing the completion notification from the queue.

Q-Stream Management

qpi\_qstream\_add()

Add an operation to the q-stream.

qpi\_qstream\_remove()

Remove an operation from the q-stream.

qpi\_qstream\_launch()

Trigger all operations in a q-stream to run.

qpi\_qstream\_abort()

Aborts operations on a stream.

qpi\_qstream\_status()

Status of all operations on the q-stream.

---

• Yes. For the SC26 proof of concept, this abstraction can make programming materially easier without
  implementing the full architecture we discussed.

  The demo should prove one claim:

  > An application supplies a quantum program; libquantum independently selects a suitable allocated device,
  > compiler path, and QRI execution route.

  It does not need to prove that every SDK works with every compiler and every device.

  ## Minimal application experience

  The default path could be:

  circuit = build_circuit_with_qiskit()

  result = libquantum.execute(circuit)

  For transparency and experimentation:

  plan = libquantum.plan(circuit)

  print(plan)
  result = libquantum.execute(plan)

  And explicit hints remain possible:

  plan = libquantum.plan(
      circuit,
      device="ibm-156",
      compiler="mqt",
  )

  That supports three useful modes:

  libquantum.execute(circuit)                         # automatic
  libquantum.execute(circuit, compiler="mqt")        # compiler selected
  libquantum.execute(circuit, device="iqm-20")       # device selected

  The explicit paths are valuable for the demo because they show that policy is controllable rather than
  hidden.

  ## Simple planning policy

  For the proof of concept, selection can be deterministic.

  First, inspect the program:

  format: qiskit-circuit
  logical_qubits: 12
  features:
    dynamic_control: false
    mid_circuit_measurement: false

  Then discover resources through QRI:

  resources:
    - id: iqm-20
      qubits: 20
      supported_compilers: [qiskit, mqt]

    - id: ibm-156
      qubits: 156
      supported_compilers: [qiskit, mqt]

  Then apply a simple rule:

  1. Remove devices with insufficient qubits.
  2. Prefer the smallest suitable device.
  3. Find compilers accepting the input format and selected device.
  4. Apply an application compiler hint when supplied.
  5. Otherwise use the configured default for that device.
  6. Compile and submit through QRI.

  For example:

   Program                                    Selected device    Default compiler
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━
   12-qubit Qiskit circuit                       IQM 20-qubit    MQT
  ─────────────────────────────────────────  ─────────────────  ──────────────────
   40-qubit Qiskit circuit                      IBM 156-qubit    Qiskit
  ─────────────────────────────────────────  ─────────────────  ──────────────────
   12-qubit circuit with compiler="qiskit"       IQM 20-qubit    Qiskit
  ─────────────────────────────────────────  ─────────────────  ──────────────────
   40-qubit circuit with device="iqm-20"       Planning error    None

  The exact preferences are policy choices. What matters is that the decision is visible and reproducible.

  ## Compiler selection must also consider input format

  Device selection can primarily use qubit count in the demo. Compiler selection needs at least two inputs:

  program format + selected device

  A small static compatibility table is enough:

  compilers:
    qiskit:
      inputs: [qiskit-circuit, qpy, openqasm-3]
      targets: [ibm-156, iqm-20]

    mqt:
      inputs: [qiskit-circuit, openqasm-2, openqasm-3]
      targets: [ibm-156, iqm-20]

    catalyst:
      inputs: [catalyst-mlir, pennylane-program]
      targets: [qri-runtime]

  This avoids claiming that Catalyst can directly consume an arbitrary Qiskit circuit. If you implement a
  Qiskit-to-Catalyst bridge for the demo, register that translation explicitly:

  qiskit-circuit
      -> OpenQASM 3
      -> Catalyst-compatible import
      -> Catalyst compilation

  The planner can then recognize it as a supported route.

  ## What to build

  I would limit the proof of concept to five components.

  ### 1. Program descriptor

  A Python object is enough initially:

  Program(
      format="qiskit-circuit",
      payload=circuit,
      logical_qubits=12,
      features={...},
  )

  The Python binding can automatically recognize supported objects:

  program = libquantum.import_program(circuit)

  Support only the formats needed for the demo:

  • Qiskit circuit or QPY;
  • OpenQASM 3;
  • Catalyst MLIR or QIR, if required.

  ### 2. Resource descriptor

  Normalize IBM and IQM information:

  Resource(
      id="iqm-20",
      qubits=20,
      native_gates=...,
      connectivity=...,
      accepted_payloads=...,
  )

  Populate it through existing QRI/QFw discovery and qhw-data records.

  ### 3. Compiler registry

  Use a static table or a few hardcoded adapter objects:

  class Compiler:
      def accepts(program, resource) -> bool: ...
      def compile(program, resource, options) -> Artifact: ...

  Implement three adapters:

  QiskitCompiler
  MQTCompiler
  CatalystCompiler

  There is no need for dynamic plugin discovery in the proof of concept.

  ### 4. Deterministic planner

  plan = planner.plan(
      program,
      resources,
      device_hint=None,
      compiler_hint=None,
  )

  The plan should contain:

  program:
    format: qiskit-circuit
    logical_qubits: 12
  resource:
    id: iqm-20
  compiler:
    id: mqt
  decision:
    resource_reason: smallest device with sufficient qubits
    compiler_reason: preferred compatible compiler for iqm-20

  ### 5. QRI executor

  The executor takes the compiled artifact and selected resource:

  job = qri.submit(
      resource=plan.resource,
      artifact=compiled,
      reservation=allocation_context,
  )

  The application should receive the same libquantum job and result interface for IBM and IQM.

  ## Keep planning separate from compilation

  Even in the proof of concept, preserve this distinction:

  plan = libquantum.plan(circuit)
  compiled = libquantum.compile(plan)
  job = libquantum.submit(compiled)
  result = job.result()

  Then offer a convenience operation:

  result = libquantum.execute(circuit)

  This makes the automatic behavior easy while letting the SC26 demo expose each decision.

  It also leaves room for circuit cutting later:

  plan
    -> choose devices
    -> partition circuit
    -> compile each partition
    -> execute tasks
    -> reconstruct result

  You do not need to implement that now.

  ## What to defer

  For this demonstration, I would defer:

  • arbitrary compiler pipeline composition;
  • dynamic compiler plugins;
  • multi-QPU entanglement;
  • general circuit cutting;
  • generic object binding;
  • mesh and stream semantics;
  • complex compiler scoring;
  • automatic compiler fallback after execution begins;
  • complete SDK and compiler cross-product support.

  A static supported-path matrix demonstrates interoperability more clearly.

  ## Does it make programming easier?

  For applications that otherwise have to discover devices, construct compiler targets, choose a transpiler,
  submit provider jobs, poll them, and normalize results, yes.

  The difference is:

  Current application responsibility:
  discover -> select -> query target -> choose compiler -> compile
  -> translate -> submit -> poll -> normalize

  versus:

  With libquantum:
  construct program -> execute

  The proof is strongest if the same application source runs these demonstrations:

  1. A small circuit automatically selects IQM.
  2. A larger circuit automatically selects IBM.
  3. A compiler hint changes Qiskit to MQT without changing execution code.
  4. An unsupported combination produces a clear planning error.
  5. The printed plan explains every decision.

  That is a focused and credible SC26 demonstration of SDK, compiler, and hardware independence.
