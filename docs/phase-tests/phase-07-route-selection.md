# Phase 7 Route Selection Gate

## Scope

This gate verifies Program Representation, Program Route, and Route Selection.
It covers `LQ-PRG-001` through `LQ-PRG-005`, `LQ-PRG-007`,
`LQ-COMP-005` through `LQ-COMP-007`, `LQ-SEL-001`, `LQ-SEL-002`,
`LQ-SEL-008`, and `LQ-DEMO-006`.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK, compiler, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

The gate combines both dummy compiler plugins, both allocated devices, SDK and
serialized dummy programs, an injected conflicting recognizer, explicit
representations, opaque payloads, compiler rejection, and capacity changes.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_07_route_selection.py
```

## Assertions and Expected Output

The gate verifies SDK, serialized, explicit, unknown, ambiguous, and opaque
imports; every constraint; capacity and compiler rejection explanations;
deterministic unranked ordering; callback applicability; context ownership;
generation rejection; repeated route opening; and opened-route stability over
refresh. Opaque queries invoke no compiler callback.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Each initialized context finalizes its dummy compilers. Dummy QPM and
reservation state remains externally owned.

## Failure Interpretation

A failure means representation identification, ownership, eligibility,
explanation, deterministic ordering, application choice, or fixed route
binding differs from the design. Correct Phase 7 before compilation and
submission work.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_07_route_selection.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `8 passed in 0.06s`
