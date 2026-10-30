# Phase 14 Query and qhw Discovery Gate

## Purpose

This isolated gate validates allocation-scoped QPM query discovery and direct
qhw record retention from Compiler Integration Amendment 02.

## Linked Design

- SDK Amendment 01 `Allocation-Scoped QPM Discovery`
- SDK Amendment 01 `Authoritative QPM Query Contract`
- Compiler Amendment 02 `Retained qhw Records`
- Compiler Amendment 02 `Discovery and Retention`

## Prerequisites

- Python 3.10 or later;
- the libquantum source tree; and
- pytest from the development environment.

The gate uses in-memory QPM and compiler fixtures. It needs no QFw service,
reservation, credential, network, SDK, or hardware.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_phase_14_query_qhw_discovery.py
```

## Assertions

The gate proves discovery of one and multiple exact reservation/service pairs,
route construction from query-advertised inputs only, unchanged qhw mapping
identity with named nodes and structured operations, refresh generation and
opened-route retention, independence from a device-type environment variable,
and removal of the separate input-capability RPC.

## Expected Output

```text
4 passed
```

The elapsed time is environment-dependent.

## Cleanup and Retained Evidence

Each initialized context is finalized and the fixtures create no external
state. Retain the pytest command, pass count, commit ID, and rendered amendment
checklist as the phase evidence.
