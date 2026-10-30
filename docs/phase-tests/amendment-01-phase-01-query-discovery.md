# Amendment 01 Phase 1 Query Discovery Gate

## Purpose

This isolated gate validates the authoritative QPM query and allocation-scoped
discovery flow in SDK Integration Amendment 01.

## Linked Design

- `Allocation-Scoped QPM Discovery`
- `Authoritative QPM Query Contract`
- `Accepted Program Inputs`
- `libquantum Initialization`

## Prerequisites

- Python 3.10 or later;
- the libquantum source tree; and
- pytest from the development environment.

The gate uses in-memory QPM and compiler fixtures. It needs no QFw service,
allocation, credential, network, SDK, or hardware.

## Command

```console
python3 -m pytest -q tests/phase_tests/test_amendment_01_phase_01_query_discovery.py
```

## Assertions

The gate proves selector-free discovery of one and multiple reserved services,
route construction from exact query inputs, absence of the removed capability
RPC, service-specific identity and mapping failures, and preservation of an
opened route's exact service, reservation, and execution binding across
refresh.

## Expected Output

```text
5 passed
```

The elapsed time is environment-dependent.

## Cleanup and Retained Evidence

Each initialized context is finalized and the fixtures create no external
state. Retain the pytest command, pass count, commit ID, and rendered amendment
checklist as the phase evidence.
