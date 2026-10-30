# Phase 11 Isolated Workflow Gate

## Scope

This gate proves the complete public application workflow with dummy boundary
integrations only. It covers application-selected routes, separate compilation
and submission, completion delivery, opaque execution, and finalization.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK backend, external compiler, credential, network
connection, reservation, simulator, or hardware is required.

## Fixture Inputs

The workflow uses compiler A with the small QPY device, compiler B with the
large OpenQASM 2 device, two reservation-bound completion queues, a later
opaque queue, successful and failed execution events, confirmed cancellation,
and a separate context with two deliberately outstanding operations.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_11_isolated_workflow.py
```

## Assertions and Expected Output

The workflow executes initialization, recognized and explicit program import,
route query and application choice, route opening, queue creation, separate
compilation, four submissions, status, cancellation, successful and failed
completion retrieval, opaque submission with no compiler call, successful
finalization, and timed-out finalization with exact outstanding QRI IDs.

The retained logical trace has this shape:

```text
initialize
import:a, import:b
open:<route-a>, open:<route-b>
compile:dummy:a, compile:dummy:b
run:<cid-a1>, run:<cid-a2>, run:<cid-b1>
status:<cid-a1>, cancel:<cid-b1>
complete:<cid-a1>, complete:<cid-a2>:failed
complete:<cid-b1>:cancelled
import:opaque, run:<cid-b2>:opaque
finalize:finalized
```

Every `run()` value equals the accepted QRI `cid` and its private operation key.
The workflow also asserts that no SDK execution backend module is imported.

## Cleanup

The successful workflow consumes every terminal event before finalization. The
timeout workflow unregisters and closes its library-owned listener without
cancelling either outstanding QPM operation.

## Failure Interpretation

A failure means the common application flow, compiler/device interoperability,
QRI identity, opaque bypass, event delivery, or lifecycle behavior differs
from the detailed design.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_11_isolated_workflow.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `2 passed in 0.05s`
