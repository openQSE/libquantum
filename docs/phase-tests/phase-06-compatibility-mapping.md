# Phase 6 Compatibility Mapping Gate

## Scope

This gate verifies Compatibility Mapping. It covers `LQ-COMP-004`,
`LQ-COMP-008`, `LQ-FMT-003`, `LQ-FMT-004`, and `LQ-CON-002`.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, SDK, compiler, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

Both manifest-loaded dummy compilers and both allocated dummy devices provide
different exact input and output representations. The unrelated device remains
outside the allocation. One test changes a QPM version and another changes
usable capacity before explicit refresh.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_06_compatibility_mapping.py
```

## Assertions and Expected Output

The gate verifies the exact compiler-input, compiler-output, device-input, and
version cross-product; input and device indexes; separate opaque definitions;
deterministic IDs; immutable mappings; generation changes; and exclusion of
unreported versions and unrelated devices. It proves map construction invokes
no program analysis, compatibility, ranking, or selection callback and that a
refresh does not mutate an earlier definition.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Each initialized context finalizes its dummy compilers. Dummy QPM and
reservation state remains externally owned.

## Failure Interpretation

A failure means exact-version intersection, structural route identity,
indexing, opaque forwarding, refresh stability, or the separation between map
construction and program analysis differs from the design. Correct Phase 6
before route selection.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_06_compatibility_mapping.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `5 passed in 0.06s`
