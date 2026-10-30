# Phase 5 Context Lifecycle Gate

## Scope

This gate verifies Initialization and Finalization. It covers `LQ-INIT-001`
through `LQ-INIT-004` and `LQ-FIN-001` through `LQ-FIN-005`.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK, compiler, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

The gate combines both manifest-loaded dummy compilers with the allocation
scoped dummy runtime and QPMs. Tests inject runtime, compiler, discovery,
operation-status, timeout, and queue-cleanup states through private test
boundaries while calling the public finalization contract.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_05_context_lifecycle.py
```

## Assertions and Expected Output

The gate verifies ordered initialization, ready-state enforcement, runtime,
plugin, and discovery failure boundaries, reverse unwind, successful operation
wait, exact timeout reporting, idempotent cleanup, and immutable nested
configuration. It proves finalization never implicitly cancels operations,
shuts down shared QPMs, or releases external allocation state.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Every initialized compiler is finalized in reverse order. Owned test queues
are closed. Dummy allocations, reservations, and QPMs remain externally owned.

## Failure Interpretation

A failure means lifecycle ordering, state enforcement, wait semantics, timeout
reporting, reverse cleanup, or external ownership differs from the detailed
design. Correct Phase 5 before compatibility-map construction.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_05_context_lifecycle.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `9 passed in 0.04s`
