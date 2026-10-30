# Amendment 02 Compiler Integration Traceability

This audit maps the compiler integration amendment to implementation ownership
and executable evidence. It covers the exact revisions recorded in the Phase 7
compiler package integration procedure.

## Traceability map

| Design responsibility | Implementation | Executable evidence |
| --- | --- | --- |
| Preserve complete qhw records and generations | `Device`, discovery, `CompilerRequest` | Core phases 13, 14, and 15 |
| Use `QPM.query()` as the input contract | QRI discovery adapter and QFw QPM query | Amendment 01 Phase 1 |
| Match exact input and output versions without callbacks | Compatibility and route-map modules | Core phases 6 and 15 |
| Analyze imported SDK objects once | Representation registry and plugin analyzers | Core Phase 15 and both plugin gates |
| Discover independent compiler packages | Filesystem manifest loader | Core Phase 3 and both plugin gates |
| Treat compiler results as authoritative | `Context.compile()` and `Context.run()` | Core phases 8 and 15 |
| Construct Qiskit targets from qhw | Public `qhw_to_qiskit_target()` helper | QFw target-helper gate and Qiskit tests |
| Compile Qiskit to the QPM contract | Qiskit compiler package | Qiskit package and Phase 7 gates |
| Construct MQT targets from qhw | Public `qhw_to_mqt_target()` helper | QFw target-helper gate and MQT tests |
| Compile native and converted MQT inputs | MQT compiler package | MQT package and Phase 7 gates |
| Keep core free of compiler SDKs | Lazy core plus external plugin packages | Core Phase 1 and core-only wheel import |
| Defer Catalyst | No Catalyst manifest or registration | Core Phase 1 import and route audits |
| Qualify deployed QFw/QPM paths | Deployment and cluster system gates | Deferred as described below |

The QFw runtime bridge lazily imports reservation and QPM resolver utilities
from `qfw_qiskit`; it does not import or construct `QFwBackend`,
`QFwBackendV2`, a target helper, or either compiler package. The core has no
Qiskit or MQT import. SDK and target-helper imports occur only inside their
independently installed compiler callbacks.

## Anti-drift results

The following checks were run from the libquantum repository:

```bash
rg -n 'get_program_input_capabilities' libquantum tests
rg -n 'from (qiskit|mqt)|import (qiskit|mqt)' libquantum
python3 -m pytest -q
```

The removed program-input RPC appears only in negative assertions. No core
module imports Qiskit or MQT. The core suite reports `121 passed, 1 skipped`.
The one skip is an environment-gated deployment test and is not counted as
qualification.

The compiler packages are separate Git repositories. Their exact matrices are
Qiskit 2.2.3 for `libquantum-compiler-qiskit` and MQT Core 4.1.1 with Qiskit
2.5.2 for `libquantum-compiler-mqt`. Neither package declares or imports the
other. QFw helper types remain inside QFw and the compiler callbacks; no
helper-specific target type appears in a libquantum public record.

## Deployment status

The deployed-QPM checklist remains open. The current evidence uses isolated
wheel installations and dummy QRI/QPM boundaries. No active QFw allocation,
selected deployed QPM, provider credentials, or retained physical-device run
was available during this gate. Simulator evidence will be collected through
the QFw-SLURM-Cluster dual-backend system qualification after `QFwBackendV2`
is implemented. Physical-device evidence must remain distinct and unchecked
until an operator provides an eligible allocation and provider access.

No Catalyst compiler is registered, and Backline direct execution is not
counted as compiler qualification.
