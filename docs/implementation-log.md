# Implementation Log

## 2026-10-29 — Python phase-gate command

During the Phase 1 gate, the implementation plan's standard command failed
before test collection because the implementation environment has no `python`
executable. The supported interpreter is `/usr/bin/python3` (Python 3.10.12),
and pytest 9.1.1 is installed for it. Running the same gate through `python3`
completed successfully with 10 passing tests.

Decision: use `python3 -m pytest` as the exact phase-gate command in this
repository and its phase-test procedures. This is an environment correction;
it does not change the designed API, test modules, assertions, or phase
boundaries.

## 2026-10-29 — Installable package baseline

The operator required libquantum to remain installable, configurable, and
runnable as implementation proceeds. The original Phase 1 checklist covered
the importable package boundary but did not explicitly require build metadata
or an installed-package gate.

Decision: maintain standard `pyproject.toml` packaging from the foundation
phase onward. Keep runtime dependencies empty so importing the installed core
does not require SDK, compiler, QFw, or DEFw packages. Configuration remains
the designed `InitConfig` application contract. Add an isolated local-install
assertion to the Phase 1 gate and keep compiler discovery restricted to its
configured paths.

The first isolated installation exposed setuptools 59.6 in the target
environment. That version accepts the PEP 517 build request but ignores PEP
621 project metadata, producing an empty `UNKNOWN-0.0.0` wheel. Add equivalent
`setup.cfg` metadata as a compatibility source for the deployed build tool.
Keep `pyproject.toml` authoritative for modern installers. The packaging gate
must use the locally available toolchain without downloading build packages.

## 2026-10-29 — Phase 8 completion-queue prerequisite

Phase 8 requires `run()` to accept a registered, route-bound completion queue,
but the plan assigns the complete queue implementation to Phase 9. Submission
cannot be validated against the designed QRI contract with an unregistered
stand-in.

Decision: implement endpoint creation, QPM registration, route ownership, and
basic close support with Phase 8 submission. Keep nonconsuming wait, event
draining, completion translation, terminal marking, failure isolation, and the
full cleanup gate in Phase 9. This preserves the public design and narrows the
Phase 8 prerequisite to the behavior needed for real QPM submission.

## 2026-10-29 — QFw completion registration scope

The frozen QFw implementation registers event endpoints by reservation and
can filter on a `cid`, but QRI assigns the `cid` only after `async_run()` has
accepted the operation. Its submission API has no completion-registration
argument, and terminal records do not retain an application queue identifier.
Registering several unfiltered endpoints for one reservation would therefore
broadcast duplicate completion events to every queue.

Decision: allow one active libquantum completion queue per QPM reservation and
share it among that reservation's independent operations. Multiple queues
remain supported across distinct route reservations. This uses QFw's direct
endpoint delivery without adding a central redistributor or a post-submission
registration race. The Phase 12 deployment gate must report same-reservation
multi-queue delivery as unsupported until QFw supplies an atomic correlation
or registration field.

## 2026-10-29 — QFw-compatible OpenQASM fixture output

The isolated workflow exposed that dummy compiler B and its target QPM both
declared OpenQASM 3 output. The reviewed QFw circuit payload implementation
accepts OpenQASM 2, QPY, compressed QPY, and Pulser representations; it does
not accept OpenQASM 3 as a direct QPM payload.

Decision: keep compiler A's OpenQASM 3 input recognizer for cross-toolchain
input coverage, but change compiler B's output and the large dummy QPM's input
declaration to `openqasm2` version `2.0`. Compiler B now returns text that the
existing QFw payload envelope can carry. This aligns isolated execution with
the production submission contract instead of adding an unsupported format.

## 2026-10-29 — Amendment 01 QPM input qualification

The amendment's IQM query example includes a native `iqm.pulla` input, while
the declared owning QFw repository currently routes every qualifying QPM
through an OpenQASM 2.0 `qasm` payload. No native Pulla or QPY submission path
exists in that repository. Its IBM-developed QRMI integration is the shim QPM,
not a separate IBM hardware provider, and the shim also consumes OpenQASM 2.0
before its provider conversion.

Decision: advertise only `openqasm` version `2.0`. Mark it non-native for IQM,
fake IQM, and the QRMI/QDMI shim, and native for the Qb, NWQ-Sim, and TNQVM
runner paths. Reject query advertisements that do not have a declared
execution path and reject submitted representation/version pairs that do not
match the advertisement. When an IQM or shim device ID is not configured,
publish an explicit unsupported qualification status instead of inventing a
device identity. This narrows qualification to implemented behavior without
changing the amendment's extensible query schema.

## 2026-10-29 — Compiler target cache generation

The compiler plan requires target caches to distinguish immutable device
generations, but the amended `CompilerRequest` carried only the device ID and
qhw records. A compiler could not construct the required cache key without
guessing a generation from provider-specific record contents or object
identity.

Decision: add the opened device's explicit `snapshot_generation` to
`CompilerRequest` and pass it unchanged with the authoritative qhw records.
Use that value in compiler target cache keys and provenance. This is the
smallest contract correction that implements the amendment's stated cache
isolation and prevents refreshed qhw data from mutating or reusing an older
opened route's target.

## 2026-10-29 — MQT compiler environment and QPM exchange path

QFw-SLURM-Cluster installs QFw with Qiskit 2.2.3 and installs MQT Core 4.1.1
in a separate environment with Qiskit 2.5.2. Direct qualification confirmed
that MQT rejects the 2.2.3 frontend, cannot emit QDMI OpenQASM 2, and emits
OpenQASM 3.1 or QIR for explicit targets. The currently qualified QFw QPMs
accept OpenQASM 2.0, not OpenQASM 3 or QIR.

Decision: qualify the MQT compiler in the cluster's separate MQT environment.
Use MQT's public target compilation, OpenQASM 3, `QCProgram`, and Qiskit export
APIs, then serialize a validated static terminal-measurement circuit as
OpenQASM 2.0. Record every conversion stage in provenance and reject programs
whose classical or parameter semantics cannot be represented honestly.
Amend the isolated integration gate to use separate exact Qiskit and MQT
environments rather than requesting an impossible co-installation of Qiskit
2.2.3 and 2.5.2.

The separate MQT environment also means the Qiskit compiler's 2.2.3 object
registration cannot supply import analysis for MQT's 2.5.2 converted input.
The core registration contract requires every registered Python type to have
an analyzer and invokes it exactly once during import.

Decision: let the MQT plugin register an equivalent non-mutating analyzer for
its exact Qiskit 2.5.2 converted-input pair. Compilation consumes the stored
requirements and never analyzes again. If a future shared registration exists
for the same exact type and version, the core's existing one-analysis import
contract remains authoritative.

## 2026-10-29 — Portable Qiskit OpenQASM 2 exchange basis

The shared compiler integration fixture caused Qiskit 2.2.3 to target the
cluster's `rz`, `sx`, `x`, `cx`, and measurement operation set. Qiskit's
OpenQASM 2 exporter emitted `sx` without a gate declaration, and the same
exact Qiskit version rejected that artifact on import. The target compilation
was valid, but the serialized exchange artifact was not portable OpenQASM 2.

Decision: preserve the target-compiled circuit and its layout as the
authoritative compiler result, but lower a serialization-only copy to the
standard `u3` and `cx` OpenQASM 2 exchange basis before export. Record
`qasm2_basis_lowering` in compiler provenance. Validate the serialized text by
round-tripping it through the exact qualified Qiskit parser before treating
the compiler package as integrated.

## 2026-10-29 — QFw execution envelope and shot propagation

The BackendV2 primitive gate exposed two adjacent contract requirements. QFw
QPMs advertise OpenQASM 2 but their execution API accepts a mapping containing
the QASM text and qubit count, not a bare string. Qiskit supplies shots at
`run()` time, after compilation, while libquantum previously forwarded only
the execution timeout to `QPM.async_run()`.

Decision: make each compiler result payload the exact QFw execution envelope
with `qasm`, `num_qubits`, `representation`, and `version`, while retaining the
OpenQASM `Representation` record. Preserve that mapping by identity through
`Context.run()`. Pass a non-null `ExecutionOptions.shots` beside the artifact
to `QPM.async_run()`, and let the QPM copy it into `num_shots` without mutating
the compiler-owned payload. Extend the QPM execution binding with the optional
`shots` argument. This retains authoritative artifact handoff and supplies the
runtime-only control at the boundary that owns execution.

## 2026-10-29 — Explicit QRI API categories

The live DEFw proxy does not reliably expose Python method annotations or
interface ownership. Inferring query, telemetry, and execution methods from a
proxy object's shape caused valid deployed services to be rejected.

Decision: bind each QRI call through its explicit API category and method name.
Treat the allocation directory entry as authoritative for the query response.
This preserves the public QRI boundary without depending on proxy reflection.

## 2026-10-29 — Allocation-scoped capacity

The deployed QPM's physical device record describes total simulator capacity,
while Slurm admission supplies the qubit limit granted to the running job. A
route built from total capacity could accept a program larger than its active
allocation.

Decision: query capacity telemetry with the active external allocation ID and
caller-owned scope. QFw returns the admitted task-class limit only when the
scope matches the allocation. Unscoped callers continue to receive no
allocation-specific capacity.

## 2026-10-29 — Exact QFw installation prefix

QFw-SLURM-Cluster can install a developer build under a nondefault prefix, but
the remote site-service shell inherited the image's default `QFW_PREFIX`.
Activation therefore started a different QFw build than the application used.

Decision: export the requested prefix in the remote service environment before
activation. System scripts also export the exact prefix and record installed
module and compiler-manifest paths before running.

## 2026-10-29 — QHW parameter metadata and active width

MQT target construction rejected the live NWQSim qhw record because `rz` did
not declare its required angle. Both compiler serializers also retained the
target's full trailing idle width, causing a two-qubit program to exceed its
two-qubit Slurm allocation after compilation.

Decision: declare the `theta` parameter for NWQSim `rz`. Trim only trailing
idle target qubits from Qiskit and MQT serialized artifacts while preserving
physical indices, interior idle qubits, topology, and layout provenance.

## 2026-10-29 — QFw result normalization

The live NWQSim QPM returns a raw counts mapping and a compressed statevector.
Qiskit adapter tests had covered the normalized envelope but not both live
forms.

Decision: normalize the qualified raw counts and statevector forms at the QFw
adapter boundary. Preserve shot totals, bit order, and experiment order, and
reject unknown forms instead of guessing.

## 2026-10-29 — Simulator parity without a seed control

The system plan requested equal seeds for legacy and V2 runs, but NWQSim's
deployed QPM contract exposes no seed option. Requiring identical random count
maps would claim a control that the selected route cannot provide.

Decision: run the same GHZ circuit, shots, allocation, and NWQSim route through
both backends. Compare backend class identity, supported operations,
transpilation, count totals and allowed keys, statevector availability,
metadata, lifecycle behavior, and cleanup. Label the retained evidence as
simulator-only and keep physical-device qualification separate.

## 2026-10-29 — Installable dual compiler environments

QFw's qualified Qiskit adapter uses Qiskit 2.2.3, while MQT Core 4.1.1 requires
the separately qualified Qiskit 2.5.2 path. One environment cannot honestly
represent both exact dependency sets.

Decision: have QFw-SLURM-Cluster build wheels for libquantum, both compiler
packages, and qfw-slurm. Install core and the Qiskit compiler into the QFw
environment, and install core plus the MQT compiler into a separate MQT
environment. Verify installed manifests and imports before any Slurm run.

## 2026-10-29 — Pytest inside the initialized DEFw process

The managed-QFw phase harness initially replaced its `defw-python` process
with `python -m pytest`. Process replacement discarded the initialized DEFw
runtime, so the new process could not find the directory service even though
the Slurm reservation and site QPM were healthy.

Decision: invoke `pytest.main()` inside the application process created by
`defw-python`. This preserves DEFw initialization while executing the same
marked test and arguments. Failed jobs 42 through 49 remain diagnostic only;
completed job 50 is the retained qualification run.
