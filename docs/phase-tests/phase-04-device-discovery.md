# Phase 4 Device Discovery Gate

## Scope

This gate verifies Device Discovery Through QRI and Device Representation. It
covers `LQ-INIT-004`, `LQ-RES-001`, `LQ-RES-002`, `LQ-DEV-001`,
`LQ-DEV-002`, `LQ-COMP-003`, `LQ-QFW-001`, `LQ-QFW-003`, and `LQ-QFW-005`.

## Prerequisites

- Installed libquantum package or repository root on the Python import path
- Python 3.10 or newer with pytest

No deployed QFw, DEFw, device service, credential, network connection,
reservation, or hardware is required.

## Fixture Inputs

The in-memory runtime contains two differently sized devices in the active
allocation and one unrelated device. Each QPM exposes versioned input
capabilities, target telemetry, capacity, execution, status, cancellation,
and event registration methods. Configurable modes cover incomplete,
unavailable, malformed, stale, and mismatched access.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_04_device_discovery.py
```

## Assertions and Expected Output

The gate verifies allocation filtering, exact capability translation, private
service and reservation binding, immutable stable snapshots, changed and stale
refresh generations, incomplete metadata, safe discovery errors, and dummy
QPM execution, status, cancellation, and terminal event behavior. The
unrelated device must never be resolved or appear in the snapshot.

The expected result is all tests passing with no skips or expected failures.

## Cleanup

The event fixture closes both pipe descriptors. All remaining fixtures are
in-memory objects released when pytest exits.

## Failure Interpretation

A failure means allocation scope, QRI field translation, snapshot identity,
private access association, error safety, or the reusable QPM fixture differs
from the detailed design. Correct Phase 4 before context lifecycle work.

## Evidence Retained

Command: `python3 -m pytest -q tests/phase_tests/test_phase_04_device_discovery.py`

Execution date: 2026-10-29

Environment: Python 3.10.12 and pytest 9.1.1

Result: `9 passed in 0.07s`
