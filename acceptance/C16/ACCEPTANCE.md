# C16 System Acceptance

Status: NOT YET EXECUTED

C16 is accepted only after both component implementations are complete and the
cross-repository handoff is demonstrated.

| ID | Requirement | Owner | Status |
|---|---|---|---|
| SYS-AT01 | C16 branches descend from frozen pre-C16 baselines | SYSTEM | PENDING |
| SYS-AT02 | Production P4/P5 selections generated without manual live-selection file | ML | PENDING |
| SYS-AT03 | Production selections deterministic | ML | PENDING |
| SYS-AT04 | Stale/missing production data fails closed | ML | PENDING |
| SYS-AT05 | Generated selections feed accepted signal-plan path | ML | PENDING |
| SYS-AT06 | Valid next-open inputs generate production order ticket | ML | PENDING |
| SYS-AT07 | ML ticket imports into watcher without editing | SYSTEM | PENDING |
| SYS-AT08 | Re-import is idempotent | WATCHER | PENDING |
| SYS-AT09 | Wrong execution date cannot become live | WATCHER | PENDING |
| SYS-AT10 | Controlled board causes accepted C15 transition | WATCHER | PENDING |
| SYS-AT11 | Durable transition event occurs exactly once | WATCHER | PENDING |
| SYS-AT12 | Watcher event traces back to ML production ticket/run | SYSTEM | PENDING |
| SYS-AT13 | Existing accepted tests remain passing | BOTH | PENDING |
| SYS-AT14 | No broker order placement exists in C16 | SYSTEM | PENDING |
| SYS-AT15 | A07 strategy/model/allocation remains frozen | SYSTEM | PENDING |

## Acceptance evidence

Actual command output and generated fixture hashes will be added only after
execution.

No item may be marked PASS based solely on implementation intent.
