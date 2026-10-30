# Phase 12 Managed QFw Integration Gate

## Status

Simulator deployment qualification completed in QFw-SLURM-Cluster Slurm job
50. Physical-device qualification remains separate and is not claimed.

## Purpose

This opt-in gate replaces dummy boundaries with an activated QFw, DEFw, and
qfw-slurm environment while retaining the Phase 11 public application flow. It
validates the managed-QRI boundary only; broader SC26 qualification remains
future design work.

## Frozen Environment Prerequisites

The operator must provide all of the following before setting the opt-in flag:

- activated QFw and DEFw Python paths from the frozen deployment;
- an allocation ID in `QFW_ALLOCATION_ID`;
- launcher-owned managed reservations in `QFW_RESERVATIONS`;
- reachable reserved QPM execution and telemetry services;
- one exact device/compiler/input combination supported by those QPM
  declarations;
- installed compiler plugin paths in `LIBQUANTUM_COMPILER_PATHS`, separated by
  the platform path separator;
- a test-input factory in `LIBQUANTUM_INTEGRATION_FACTORY` using
  `module:callable` syntax;
- a JSON object in `LIBQUANTUM_QFW_COMPONENT_REVISIONS` with nonempty `QFw`,
  `DEFw`, `qfw-slurm`, and `compiler` revisions;
- `LIBQUANTUM_QFW_EVIDENCE_KIND` set to `simulator` or `physical-device`;
- site credentials supplied through the normal QFw mechanism, never in test
  source, command output, or retained logs; and
- explicit authorization to submit and cancel one managed operation.

Only after those values are reviewed should the operator set
`LIBQUANTUM_QFW_INTEGRATION=1`.

## Input Factory Contract

The configured callable takes no arguments and returns a mapping containing:

- `payload`, the supported SDK object, serialized value, or opaque payload;
- `constraints`, exact `RouteConstraints` that produce one route;
- optional `representation`, `requirements`, and `ExecutionOptions`;
- optional `opaque=True`; and
- optional positive `completion_timeout`, defaulting to 300 seconds.

The factory is deployment configuration, not a runtime plugin installation
mechanism in libquantum.

## Command

```console
python3 -m pytest -q -m qfw_integration tests/phase_tests/test_phase_12_qfw_integration.py
```

## Procedure and Assertions

The gate initializes through the active QFw adapter, discovers only the active
allocation, imports the configured input, applies exact application
constraints, opens the sole eligible route, registers a completion endpoint,
compiles unless the input is explicitly opaque, submits through managed QPM,
checks exact `cid` identity through status, requests cancellation, waits for
the terminal event, drains the selected queue, unregisters the listener, and
finalizes without releasing launcher-owned allocation or reservation state.

Successful evidence must retain sanitized logs, the component-revision JSON,
the evidence kind, discovered service/device identities, selected route ID,
accepted representation/version, operation ID, terminal outcome, and listener
cleanup result. It must not retain credentials, tokens, service objects, or
server tracebacks.

## Cleanup and Recovery

The gate requests cancellation for the accepted test operation and waits for
its terminal event. Normal cleanup closes the library-owned queue and context.
It never shuts down QPM, releases reservations, or releases the allocation.

If submission is accepted but the client loses certainty, do not retry on a
different route. Use the owning QPM and reservation to inspect the returned or
retained task identity, then perform site-approved cleanup. If no `cid` was
returned, retain sanitized QFw logs and stop for operator reconciliation.

## Known Unsupported Combination

The frozen QFw event API cannot atomically bind a newly assigned `cid` to a
pre-existing listener. libquantum therefore supports one active completion
queue per QPM reservation, shared by that reservation's operations. A same-
reservation multi-queue test must be reported as unsupported until QFw exposes
an atomic submission correlation or registration field. No central event
redistributor or automatic fallback is added.

## Current Evidence

Execution date: 2026-10-29

QFw-SLURM-Cluster revision `13e4d8331a94` runs this gate inside the initialized
DEFw Slurm application step. The frozen source revisions are QFw
`91be0828ea91`, DEFw `87b8090cd30b`, qfw-slurm `e940e0a60022`, libquantum
`f0b5f5c`, and the Qiskit compiler `b1d19e927803`.

NWQSim job 50 completed with exit code 0 in 20 seconds. Pytest reported
`1 passed, 4 warnings in 12.87s`. The retained log is
`/workspace/home/user-a/libquantum-phase12.50.out` and contains no credential
or provider secret. This is simulator evidence only.
