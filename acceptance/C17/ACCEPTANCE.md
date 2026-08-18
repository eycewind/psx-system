# C17 System Acceptance

Status: NOT YET EXECUTED

| ID | Requirement | Owner | Status |
|---|---|---|---|
| SYS-AT01 | C17 branches descend from accepted C16 merged baseline | SYSTEM | PENDING |
| SYS-AT02 | Authoritative EOD update is idempotent | WATCHER | PENDING |
| SYS-AT03 | Adjustment stage completes and verifies cleanly | WATCHER | PENDING |
| SYS-AT04 | Missing/stale signal-date EOD fails closed | WATCHER | PENDING |
| SYS-AT05 | Phase A requires no execution-session open | ML | PENDING |
| SYS-AT06 | Phase A deterministically emits frozen decision artifact | ML | PENDING |
| SYS-AT07 | Phase A preserves accepted P4/P5/allocation policy | ML | PENDING |
| SYS-AT08 | Execution date is explicit and holiday/weekend safe | SYSTEM | PENDING |
| SYS-AT09 | Live opening data is sourced from execution session, not future EOD | WATCHER | PENDING |
| SYS-AT10 | Missing/stale/wrong-date open data fails closed | WATCHER | PENDING |
| SYS-AT11 | Phase B consumes frozen Phase-A decision without recomputing signals | ML | PENDING |
| SYS-AT12 | Phase B emits canonical watcher JSON directly | ML | PENDING |
| SYS-AT13 | Automatic watcher import accepts JSON unchanged | SYSTEM | PENDING |
| SYS-AT14 | Repeated Phase A / Phase B / import is idempotent | SYSTEM | PENDING |
| SYS-AT15 | Conflicting same-session ticket fails explicitly | WATCHER | PENDING |
| SYS-AT16 | Successful workflow produces durable operational status/notification | WATCHER | PENDING |
| SYS-AT17 | Failed/blocked workflow produces visible operational failure status | WATCHER | PENDING |
| SYS-AT18 | Controlled execution-session board reaches accepted C15 transition | WATCHER | PENDING |
| SYS-AT19 | Full ticket/event provenance traces back to frozen Phase-A artifact | SYSTEM | PENDING |
| SYS-AT20 | Existing accepted regression suites remain passing | BOTH | PENDING |
| SYS-AT21 | No StockIntel/broker/order-submission capability introduced | SYSTEM | PENDING |
| SYS-AT22 | A07 strategy/model/allocation remains frozen | SYSTEM | PENDING |

## Required system demonstration

The final acceptance run must demonstrate:

    official/controlled EOD
        -> adjustment verification
        -> Phase A
        -> frozen decision artifact
        -> execution-session live open
        -> Phase B
        -> canonical JSON ticket
        -> watcher import
        -> controlled C15 state transition

The same inputs must also be rerun to prove idempotency.

Negative cases must include stale/wrong-date/missing input behavior.

No item may be marked PASS solely from implementation intent.
