# Phase 3 Compiler Integration Gate

## Scope

This gate verifies the Compiler Integration Contract design section and
`LQ-CMP-001` through `LQ-CMP-005`, `LQ-COMP-001`, `LQ-COMP-002`,
`LQ-EXT-006`, and `LQ-CON-007`.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No external SDK, compiler, QFw service, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

Two repository-owned compiler plugins are loaded through `compiler.json` and
their named factories. Dummy A has native object and converted serialized
inputs; dummy B has a different exact object version and converted input.
Both record every callback and return deterministic marker payloads.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_03_compiler_integration.py
```

## Assertions and Expected Output

The gate verifies manifest loading, exact capabilities, native and converted
classification, version-preserving object registration, serialized
recognition, deterministic output, callback propagation, reverse cleanup, and
configured rejection, exception, unavailable, and cleanup failures. It also
rejects duplicate or unknown IDs, malformed or unsupported manifests, missing
factories, registration mismatch, invalid versions, and invalid preferences.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

Every successfully initialized dummy integration is finalized in reverse
order. pytest removes mutated manifest copies and other temporary files.

## Failure Interpretation

A failure means the plugin boundary, capability invariants, representation
registry, deterministic dummy behavior, or lifecycle cleanup differs from the
approved compiler contract. Correct Phase 3 before device integration.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_03_compiler_integration.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `12 passed in 0.11s`
