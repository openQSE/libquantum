# Phase 8 Compilation and Execution Gate

## Scope

This gate verifies separate route-bound compilation, submission-only execution,
QRI operation identity, and QPM-backed status and cancellation control.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK, compiler, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

The gate uses both dummy compilers, two route-bound dummy QPMs, registered dummy
event endpoints, recognized SDK programs, opaque payloads, and injected
compiler and QPM failures.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_08_compilation_execution.py
```

## Assertions and Expected Output

The gate verifies exact compiler state and request propagation, unchanged
compiler result identity, absence of submission during compilation, QFw payload
preparation, opaque payload identity, absence of compiler work during `run()`,
exact QRI `cid` identity, operation ownership, several outstanding operations,
status and cancellation dispatch, nonterminal cancellation requests, and safe
failure translation without phantom operation records.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Each context marks accepted dummy operations terminal, unregisters event
endpoints, closes local queues, and finalizes compiler state.

## Failure Interpretation

A failure means compile and submission separation, payload preparation, route
or queue ownership, operation identity, QPM control, or a public failure
boundary differs from the design. Correct Phase 8 before queue delivery work.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_08_compilation_execution.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `7 passed in 0.06s`
