# Phase 2 Data Model Gate

## Scope

This gate verifies the Shared Data Model, Program Representation, Device
Representation, Program Route, and Operation Records sections of the detailed
design. It covers `LQ-PRG-001` through `LQ-PRG-005`, `LQ-PRG-007`,
`LQ-CON-006`, `LQ-DEV-001` through `LQ-DEV-004`, `LQ-FMT-005`, and
`LQ-OBS-002`.

## Prerequisites

- Python 3.10 or newer
- pytest available to the active Python interpreter
- repository root as the working directory

No SDK, compiler, QFw, DEFw, device service, credential, network connection,
allocation, or reservation is required.

## Fixture Inputs

The module constructs repository-owned value records, one dummy public device
snapshot, private dummy QPM bindings, opaque route definitions, and route
instances through the internal context factory.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_02_data_model.py
```

## Assertions and Expected Output

The gate verifies immutable records and mapping snapshots, designed defaults,
exact-version behavior, original payload identity, private QPM access
separation, shared device references, and distinct route instance identity. It
also verifies explicit failures for malformed identifiers, versions,
capacities, execution options, reservations, forged handles, and mutation
attempts.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

The gate creates only in-memory fixtures and no persistent resources.

## Failure Interpretation

A failure means a public value contract, ownership boundary, identity rule, or
immutability guarantee differs from the approved design. Correct the Phase 2
model before starting compiler integration work.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_02_data_model.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `18 passed in 0.04s`
