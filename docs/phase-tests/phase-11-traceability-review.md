# Phase 11 Traceability and Drift Review

## Review Result

The requirements artifact contains 154 unique requirement IDs. The detailed
design classifies 94 as part of the Python prototype and 60 as future work.
The sets are disjoint and exhaustive: no requirement is unclassified.

Every current design section maps to an implementation-plan unit and an
executable phase gate. Deployed QFw validation remains intentionally pending
in Phase 12 and is not claimed by this isolated review.

## Current Prototype Requirements

Each row lists every current requirement ID in that area. Phase evidence names
the primary gate; cross-cutting behavior is also covered by Phase 11.

| Requirement IDs | Design section | Units | Primary evidence |
| --- | --- | --- | --- |
| `LQ-INT-001`, `LQ-INT-002`, `LQ-INT-003`; `LQ-APP-001`, `LQ-APP-002` | Purpose, Application Workflow, Application API | `PH1-C1`, `PH11-C1` | Phases 1 and 11 |
| `LQ-PRG-001`, `LQ-PRG-002`, `LQ-PRG-003`, `LQ-PRG-004`, `LQ-PRG-005`, `LQ-PRG-007`, `LQ-PRG-008` | Program Representation | `PH2-C1`, `PH7-C1`, `PH8-C1`, `PH8-C2` | Phases 2, 7, 8, and 11 |
| `LQ-DEV-001`, `LQ-DEV-002`, `LQ-DEV-003`, `LQ-DEV-004` | Device Representation | `PH2-C2`, `PH4-C1`, `PH4-C2` | Phases 2 and 4 |
| `LQ-CMP-001`, `LQ-CMP-002`, `LQ-CMP-003`, `LQ-CMP-004`, `LQ-CMP-005` | Compatibility Mapping | `PH6-C1`, `PH6-C2` | Phase 6 |
| `LQ-COMP-001`, `LQ-COMP-002`, `LQ-COMP-003`, `LQ-COMP-004` | Compiler Integration Contract, Compatibility Mapping | `PH3-C1`, `PH3-C2`, `PH6-C1` | Phases 3 and 6 |
| `LQ-COMP-005`, `LQ-COMP-006`, `LQ-COMP-007` | Route Selection | `PH7-C1`, `PH7-C3` | Phase 7 |
| `LQ-COMP-008`, `LQ-COMP-009` | Compilation and Execution | `PH8-C1`, `PH8-C2`, `PH8-C3` | Phases 8 and 11 |
| `LQ-SEL-001`, `LQ-SEL-002`, `LQ-SEL-003`, `LQ-SEL-008` | Route Selection | `PH7-C1`, `PH7-C2`, `PH7-C3` | Phase 7 |
| `LQ-TASK-001`, `LQ-TASK-002`, `LQ-TASK-003`, `LQ-TASK-004`, `LQ-TASK-005` | Compilation and Execution | `PH8-C1`, `PH8-C2`, `PH8-C3` | Phases 8, 9, and 11 |
| `LQ-CQ-001`, `LQ-CQ-002`, `LQ-CQ-003`, `LQ-CQ-004`, `LQ-CQ-005`, `LQ-CQ-006`, `LQ-CQ-007`; `LQ-RSLT-001` | Completion Queues | `PH9-C1`, `PH9-C2` | Phases 9 and 11 |
| `LQ-INIT-001`, `LQ-INIT-002`, `LQ-INIT-003`, `LQ-INIT-004`; `LQ-FIN-001`, `LQ-FIN-002`, `LQ-FIN-003`, `LQ-FIN-004`, `LQ-FIN-005` | Initialization and Finalization | `PH5-C1`, `PH5-C2` | Phases 5 and 11 |
| `LQ-RES-001`, `LQ-RES-002` | Device Discovery Through QRI | `PH4-C1`, `PH4-C2`, `PH5-C1` | Phases 4 and 5 |
| `LQ-OBS-001`, `LQ-OBS-002`, `LQ-OBS-004`, `LQ-OBS-005` | Errors, Results, and Provenance | `PH10-C1`, `PH10-C2` | Phase 10 |
| `LQ-FMT-002`, `LQ-FMT-003`, `LQ-FMT-004`, `LQ-FMT-005`, `LQ-FMT-006` | Device Discovery, Operation Records, Compilation and Execution | `PH2-C2`, `PH4-C2`, `PH6-C1`, `PH8-C1`, `PH8-C2` | Phases 4, 6, 8, and 11 |
| `LQ-QFW-001`, `LQ-QFW-002`, `LQ-QFW-003`, `LQ-QFW-005`, `LQ-QFW-006` | Device Discovery, Initialization, Execution, Completion Queues | `PH4-C2`, `PH5-C1`, `PH8-C2`, `PH9-C1`, `PH12-C1` | Isolated Phases 4, 5, 8, and 9; deployment pending Phase 12 |
| `LQ-ADAPT-001`, `LQ-ADAPT-002`, `LQ-ADAPT-003`, `LQ-ADAPT-004`, `LQ-ADAPT-005`, `LQ-ADAPT-006`, `LQ-ADAPT-008` | Device Discovery, Initialization, Execution | `PH4-C2`, `PH5-C1`, `PH8-C2`, `PH12-C1` | Isolated Phases 4, 5, and 8; deployment pending Phase 12 |
| `LQ-ENV-001`, `LQ-ENV-002`, `LQ-ENV-003`, `LQ-ENV-007` | Purpose, Initialization, Execution, Completion Queues | `PLAN-PACKAGING`, `PH5-C1`, `PH8-C2`, `PH9-C1`, `PH12-C1` | Packaging and isolated Phases 5, 8, and 9; deployment pending Phase 12 |
| `LQ-CON-001`, `LQ-CON-002`, `LQ-CON-003`, `LQ-CON-006`, `LQ-CON-007` | Purpose, Device Discovery, Program Route, Execution | `PH2-C2`, `PH4-C2`, `PH7-C2`, `PH8-C2`, `PH10-C1` | Phases 2, 4, 7, 8, 10, and 11 |
| `LQ-DEMO-003`, `LQ-DEMO-006`, `LQ-DEMO-007` | Device Discovery, Route Selection | `PH4-C2`, `PH7-C1`, `PH7-C3` | Phases 4 and 7 |
| `LQ-DEMOINT-004`, `LQ-DEMOINT-005` | Purpose, Application Workflow | `PH8-C3`, `PH11-C1` | Phases 8 and 11 |
| `LQ-EXT-006` | Compiler Integration Contract | `PH3-C1`, `PH3-C2` | Phase 3 |

## Future Requirements

These 60 IDs are classified as future by the detailed design and have no
prototype implementation claim.

| Area | Requirement IDs |
| --- | --- |
| Programs and selection | `LQ-PRG-006`, `LQ-PRG-009`; `LQ-SEL-004`, `LQ-SEL-005`, `LQ-SEL-006`, `LQ-SEL-007` |
| Policies | `LQ-POL-001`, `LQ-POL-002`, `LQ-POL-003`, `LQ-POL-004`, `LQ-POL-005`, `LQ-POL-006`, `LQ-POL-007`, `LQ-POL-008` |
| Graph and executor | `LQ-GRF-001`, `LQ-GRF-002`, `LQ-GRF-003`, `LQ-GRF-004`; `LQ-EXE-001`, `LQ-EXE-002`, `LQ-EXE-003`, `LQ-EXE-004`, `LQ-EXE-005` |
| Results and observation | `LQ-RSLT-002`, `LQ-RSLT-003`, `LQ-RSLT-004`, `LQ-RSLT-005`, `LQ-RSLT-006`; `LQ-OBS-003` |
| Native and compiler extensions | `LQ-EXT-001`, `LQ-EXT-002`, `LQ-EXT-003`, `LQ-EXT-004`, `LQ-EXT-005` |
| Constraints and environment | `LQ-CON-004`, `LQ-CON-005`; `LQ-ENV-004`, `LQ-ENV-005`, `LQ-ENV-006` |
| Formats and QFw | `LQ-FMT-001`; `LQ-QFW-004`; `LQ-ADAPT-007`, `LQ-ADAPT-009`, `LQ-ADAPT-010` |
| SC26 devices | `LQ-DEMO-001`, `LQ-DEMO-002`, `LQ-DEMO-004`, `LQ-DEMO-005` |
| SC26 applications | `LQ-DEMOINT-001`, `LQ-DEMOINT-002`, `LQ-DEMOINT-003`, `LQ-DEMOINT-006` |
| Qualification | `LQ-QUAL-001`, `LQ-QUAL-002`, `LQ-QUAL-003`, `LQ-QUAL-004`, `LQ-QUAL-005`, `LQ-QUAL-006`, `LQ-QUAL-007`, `LQ-QUAL-008` |

## Design Coverage

| Non-future design section | Implementation units | Phase gates |
| --- | --- | --- |
| Application API and Workflow | `PH1-C1`, `PH11-C1` | 1 and 11 |
| Shared Data Model and Program Representation | `PH2-C1` | 2 |
| Device Representation and Program Route | `PH2-C2`, `PH7-C2` | 2 and 7 |
| Compiler Integration Contract | `PH3-C1` | 3 |
| Device Discovery Through QRI | `PH4-C1`, `PH4-C2` | 4 |
| Initialization and Finalization | `PH5-C1` | 5 |
| Compatibility Mapping | `PH6-C1` | 6 |
| Route Selection | `PH7-C1`, `PH7-C2` | 7 |
| Compilation and Execution | `PH8-C1`, `PH8-C2` | 8 and 11 |
| Completion Queues | `PH9-C1` | 9 and 11 |
| Errors, Results, and Provenance | `PH10-C1` | 10 |

The Phase 1 export-surface assertion and the final `libquantum.__all__` review
confirm that the prototype adds no graph/executor, automatic ranking policy,
compiler pass configuration, result normalization, runtime plugin installer,
or claimed deployed-hardware API.

## Artifact Discrepancies and Known Gaps

The detailed design marks 26 requirements as future even though the
requirements artifact labels them SC26: `LQ-ADAPT-007`, `LQ-ADAPT-009`,
`LQ-ADAPT-010`, `LQ-DEMO-001`, `LQ-DEMO-002`, `LQ-DEMO-004`, `LQ-DEMO-005`,
`LQ-DEMOINT-001`, `LQ-DEMOINT-002`, `LQ-DEMOINT-003`, `LQ-DEMOINT-006`,
`LQ-ENV-004`, `LQ-ENV-005`, `LQ-ENV-006`, `LQ-FMT-001`, `LQ-QFW-004`,
`LQ-QUAL-001`, `LQ-QUAL-002`, `LQ-QUAL-003`, `LQ-QUAL-004`, `LQ-QUAL-005`,
`LQ-QUAL-006`, `LQ-QUAL-007`, `LQ-QUAL-008`, `LQ-RSLT-002`, and
`LQ-RSLT-003`. Operator resolution requires either a later design expansion or
a requirements scope correction; this prototype does not claim them.

The frozen QFw API cannot atomically associate a pre-existing listener with a
new `cid`, so libquantum permits one active queue per QPM reservation. This gap
is recorded in `implementation-log.md` and must be reported by Phase 12.

Real compiler packages and application integrations are follow-on work. Phase
3 validates the plugin contract with dummy compilers only. No production
Qiskit, MQT, PennyLane, Catalyst, or other compiler integration is claimed.

The managed-QFw deployment gate remains pending because this workspace has no
operator-supplied frozen allocation, reservation, services, compilers, or
credentials. A Phase 12 safe skip is not qualification evidence.

## Classification Audit

The following read-only audit extracted unique requirement IDs from the
requirements artifact and the design's Future Requirements section, then used
set difference and intersection checks:

```console
rg -o '`LQ-[A-Z0-9-]+`' docs/requirements.md | tr -d '`' | sort -u
sed -n '1091,$p' docs/detailed-design.md | rg -o 'LQ-[A-Z0-9-]+' | sort -u
```

Result: 154 total, 94 current, 60 future, 0 overlap, and 0 unclassified.

## Individual Phase-Gate Rerun

All commands were run individually on 2026-10-29 with Python 3.10.12 and
pytest 9.1.1.

| Gate | Result |
| --- | --- |
| `test_phase_00_packaging.py` | `2 passed in 1.20s` |
| `test_phase_01_application_api.py` | `10 passed in 0.10s` |
| `test_phase_02_data_model.py` | `18 passed in 0.04s` |
| `test_phase_03_compiler_integration.py` | `12 passed in 0.06s` |
| `test_phase_04_device_discovery.py` | `9 passed in 0.04s` |
| `test_phase_05_context_lifecycle.py` | `9 passed in 0.06s` |
| `test_phase_06_compatibility_mapping.py` | `5 passed in 0.06s` |
| `test_phase_07_route_selection.py` | `8 passed in 0.05s` |
| `test_phase_08_compilation_execution.py` | `7 passed in 0.07s` |
| `test_phase_09_completion_queues.py` | `7 passed in 0.07s` |
| `test_phase_10_errors_results.py` | `8 passed in 0.06s` |
| `test_phase_11_isolated_workflow.py` | `2 passed in 0.05s` |
