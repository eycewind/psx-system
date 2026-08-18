# C16 System Acceptance

Status: **ACCEPTED**

C16 closes the production A07 signal-to-live-order handoff:

production PSX EOD data
→ frozen accepted ML model
→ current P4/P5 selections
→ accepted A07 signal plan
→ next-session order ticket
→ canonical watcher JSON handoff
→ watcher import
→ C15 live-order state machine
→ durable transition event

No broker order placement is part of C16.

## Accepted component revisions

| Component | Revision |
|---|---|
| psx-ml-research | `24add51bae3dd2f4469aa9edff724ef13dc95ead` |
| stock-watcher | `fa282d59e6ad3b1e623c2ee178b66ff8fd7ba28c` |

Pre-C16 frozen baselines remain documented in `acceptance/PRE-C16-BASELINE.md`.

## System acceptance matrix

| ID | Requirement | Owner | Status |
|---|---|---|---|
| SYS-AT01 | C16 branches descend from frozen pre-C16 baselines | SYSTEM | PASS |
| SYS-AT02 | Production P4/P5 selections generated without manual live-selection file | ML | PASS |
| SYS-AT03 | Production selections deterministic | ML | PASS |
| SYS-AT04 | Stale/missing production data fails closed | ML | PASS |
| SYS-AT05 | Generated selections feed accepted signal-plan path | ML | PASS |
| SYS-AT06 | Valid next-open inputs generate production order ticket | ML | PASS |
| SYS-AT07 | ML ticket imports into watcher without editing | SYSTEM | PASS |
| SYS-AT08 | Re-import is idempotent | WATCHER | PASS |
| SYS-AT09 | Wrong execution date cannot become live | WATCHER | PASS |
| SYS-AT10 | Controlled board causes accepted C15 transition | WATCHER | PASS |
| SYS-AT11 | Durable transition event occurs exactly once | WATCHER | PASS |
| SYS-AT12 | Watcher event traces back to ML production ticket/run | SYSTEM | PASS |
| SYS-AT13 | Existing accepted tests remain passing | BOTH | PASS |
| SYS-AT14 | No broker order placement exists in C16 | SYSTEM | PASS |
| SYS-AT15 | A07 strategy/model/allocation remains frozen | SYSTEM | PASS |

## Production acceptance run

Real PSX EOD data was used for:

- signal date: `2026-08-13`
- execution date: `2026-08-17`
- allocation: `A07_P4_25_P5_75`
- controlled acceptance account cash: PKR 50,000
- existing positions: none

The date pair intentionally crosses the 2026-08-14 market holiday and weekend.

Adjusted EOD data was verified before execution:

- `adj_apply.py --verify`
- result: `Idempotency check: 0 mismatches`
- 2026-08-13: no missing `close_adj`
- 2026-08-17: no missing `open_adj`

The ML production run produced:

- features: 303 rows
- predictions: 303 rows
- P4 selections: 3
- P5 selections: 15
- P4/P5 overlap: 1 symbol
- combined selections: 18 rows
- signal plan: 17 rows
- order ticket: 17 rows

Frozen production model:

- model: `rank_5_B_market_context_fold_2025_lightgbm_cpu.txt`
- SHA-256: `ecc95b9d78aa4dd26b30dbe4560eec716d4f21a8e190e59ea02b84a75d3643d5`
- retrained: false

Canonical watcher-facing artifact:

- file: `evidence/live/2026-08-13/order_ticket_2026-08-17.json`
- top-level JSON type: list
- rows: 17
- SHA-256: `683439c19dbd54f0766682a2c91ee1aafafed157b0142ef0fc47a8bee3b7de02`

The JSON ticket was passed to watcher unchanged.

## Cross-repository handoff evidence

First watcher import:

- inserted: true
- rows: 17
- run_id: `3b9836751925d768d3845ac937f94fa60dcd047a5ec7baa62274133300193e82`

Identical second import:

- inserted: false
- rows: 17
- run_id unchanged

Database state after repeated import:

- `a07_order_runs`: 1 row
- `a07_orders`: 17 rows

This proves the ML-to-watcher handoff is directly compatible and idempotent.

## Live-state-machine evidence

Watcher C16/C15 controlled-board evidence:

- `wrong_date_events=0`
- `valid_date_events=1`
- `repeat_events=0`
- `durable_event_count=1`

Observed accepted transition:

`READY -> ACTIONABLE`

Provenance evidence:

`('ABOT', 'READY', 'ACTIONABLE', 'c16-evidence-board', 'ACTIONABLE', 'tests/fixtures/a07_order_ticket_2026-08-11.json', 17)`

This proves:

- execution-date gating prevents wrong-session activation;
- valid execution-date processing advances the existing C15 state machine;
- durable transition history is created;
- repeated equivalent processing does not create a duplicate logical event;
- event provenance joins back through order and imported run.

## Regression evidence

### psx-ml-research

Environment:

- Python: `/home/ata/miniconda3/envs/psx-ml-research/bin/python`
- version: Python 3.12.12

C16-relevant suite command:

    pytest -q \
      tests/live \
      tests/c10/test_p4_selection.py \
      tests/c10/test_p5_selection.py \
      tests/c10/test_p4_c10_integration.py \
      tests/c10/test_p5_c10_integration.py \
      tests/c11/test_live_orders.py \
      tests/c11/test_capital_allocation.py

Result:

`34 passed`

The complete historical test suite is not currently executable in the intended
environment because `torch` is not installed. This is a pre-existing environment
dependency and no C16 failure was observed. Torch was not installed merely to
close C16.

### stock-watcher

Environment:

- Python: `/home/ata/miniconda3/envs/binbot/bin/python`
- version: Python 3.10.19

Targeted C16:

`5 passed`

Full watcher suite:

`28 passed`

No watcher production runtime changes were required during final C16 acceptance.

## Policy freeze

C16 does not change:

- accepted A07 allocation `A07_P4_25_P5_75`;
- accepted P4/P5 strategy behavior;
- frozen production model;
- order-construction policy;
- C15 live-order state-machine semantics;
- broker execution policy.

C16 does not place orders through StockIntel or any broker API.

## Retained evidence

The complete acceptance artifact set is retained under:

`acceptance/C16/evidence/live/2026-08-13/`

It includes:

- `features.parquet`
- `predictions.parquet`
- `scoring_manifest.json`
- `selections.parquet`
- `selection_manifest.json`
- `signal_plan.parquet`
- `order_ticket_2026-08-17.parquet`
- `order_ticket_2026-08-17.json`
- `production_manifest.json`

These files are the frozen evidence from the accepted production handoff run.
