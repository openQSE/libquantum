# Phase 13 Amended Contracts Gate

## Purpose

This isolated gate validates the public and plugin-facing contract changes in
Compiler Integration Amendment 02 before query-driven discovery is changed.

## Linked Design

- `Retained qhw Records`
- `Extracted Device Summary`
- `Discrete Version Registration`
- `Program Import Analysis`
- `Compiler Callback Contract`

## Prerequisites

- Python 3.10 or later;
- the libquantum source tree; and
- pytest from the development environment.

No SDK, compiler distribution, QFw installation, network, credential,
allocation, or hardware is required.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_13_amended_contracts.py
```

## Assertions

The gate proves that qhw device, coupling, and calibration mappings retain
object identity through `Device` and `CompilerRequest`, including named nodes,
structured operations, metadata, extensions, and raw provider payloads. It
also proves exact version membership, import-analyzer registration, removal of
the lossy coupling and route-compatibility records, the reduced compiler
callback protocol, and dependency-free core import.

## Expected Output

```text
5 passed
```

The elapsed time is environment-dependent.

## Cleanup and Retained Evidence

The gate creates no persistent runtime state and requires no cleanup. Retain
the pytest command, pass count, commit ID, and rendered amendment checklist as
the phase evidence.
