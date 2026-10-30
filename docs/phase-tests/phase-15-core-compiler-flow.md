# Phase 15 Core Compiler Flow Gate

## Purpose

This isolated gate validates one-time program analysis, mechanical route
filtering, exact qhw compiler dispatch, and authoritative compiler artifact
handoff from Compiler Integration Amendment 02.

## Linked Design

- `Program Import Analysis`
- `Route Construction`
- `Compiler Callback Contract`
- `Artifact Submission`
- `Refresh, Concurrency, and Lifecycle`

## Prerequisites

- Python 3.10 or later;
- the libquantum source tree; and
- pytest from the development environment.

The gate uses in-memory compiler and QPM fixtures. It needs no SDK, QFw
service, reservation, credential, network, or hardware.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_15_core_compiler_flow.py
```

## Assertions

The gate proves that SDK analysis runs once at import, route listing invokes no
compiler callback, capacity filtering rejects an oversized program, exact
output/input versions construct routes, compilation receives the identical qhw
records retained at discovery, compiler results remain authoritative, payloads
pass directly to QRI, and refresh does not replace qhw records bound to an
opened route.

## Expected Output

```text
5 passed
```

The elapsed time is environment-dependent.

## Cleanup and Retained Evidence

Each test completes in-memory operations and finalizes its context. Retain the
pytest command, pass count, commit ID, and rendered amendment checklist as the
phase evidence.
