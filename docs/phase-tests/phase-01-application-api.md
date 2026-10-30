# Phase 1 Application API Gate

## Scope

This gate verifies the Application API and Application Workflow sections of
the detailed design. It covers `LQ-INT-002`, `LQ-INT-003`, `LQ-APP-001`, and
`LQ-APP-002`.

## Prerequisites

- Python 3.10 or newer
- pytest available to the active Python interpreter
- repository root as the working directory

No SDK, compiler, QFw, DEFw, device service, credential, network connection,
allocation, or reservation is required.

## Fixture Inputs

The test imports the repository-owned `libquantum` package in the test process
and in a clean child interpreter whose import hook rejects optional SDK, QFw,
and DEFw packages.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_01_application_api.py
```

## Assertions and Expected Output

The gate verifies:

- the package imports without optional runtime dependencies;
- the exported application surface matches the design;
- lifecycle, context, route, and completion-queue signatures match the design;
- `run()` returns `str`, while status and cancellation accept `op_id`;
- context-owned handle constructors reject application construction;
- unimplemented entry points fail explicitly; and
- future graph, executor, result-processing, compiler-configuration,
  runtime-installation, and automatic-selection APIs are absent.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

The gate creates no persistent resources. pytest removes its normal temporary
state after the process exits.

## Failure Interpretation

A failure means the import boundary, public symbol set, callable signature,
handle ownership, placeholder behavior, or prototype scope differs from the
approved design. Correct the Phase 1 API contract before starting Phase 2.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_01_application_api.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `10 passed in 0.10s`
