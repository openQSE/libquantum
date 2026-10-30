# Phase 9 Completion Queue Gate

## Scope

This gate verifies direct route-bound QPM completion delivery, nonconsuming
waiting, batch retrieval, operation isolation, and listener cleanup.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK, compiler, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

The gate uses two dummy QPM reservations, multiple event endpoints, shared
queues, successful payload objects, failed and timed-out operations, confirmed
cancellation, unknown events, and injected endpoint transport failures.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_09_completion_queues.py
```

## Assertions and Expected Output

The gate verifies route registration, file descriptors, nonconsuming waits,
timeouts, batch draining, exact QRI `cid` identity, unchanged payload identity,
per-operation failure isolation, terminal tracking, multiple queues across
reservations, shared queues, unknown-event isolation, endpoint failures,
idempotent close, finalization cleanup, and absence of implicit cancellation.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Each test closes its context. Cleanup unregisters QPM listeners and closes
library-owned local endpoints without cancelling accepted operations.

## Failure Interpretation

A failure means QRI event routing, operation correlation, payload ownership,
failure isolation, listener ownership, or cleanup differs from the design or
the recorded frozen-QFw registration constraint.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_09_completion_queues.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `7 passed in 0.05s`
