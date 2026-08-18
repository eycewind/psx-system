# C17 MASTER CONTRACT
## Daily Production Orchestration

Status: DRAFT / NOT ACCEPTED

C17 operationalizes the production path accepted by C16.

C16 proved that real PSX EOD data can produce current P4/P5 selections,
an accepted A07 signal plan, a canonical JSON order ticket, and an
idempotently imported watcher run that progresses through the existing
C15 live-order state machine.

C17 removes the remaining manual shell orchestration around that path.

---

# 1. Objective

Provide a deterministic, fail-closed daily production workflow:

AFTER MARKET CLOSE

official PSX EOD
-> adjusted daily data
-> production ML scoring
-> current P4/P5 selections
-> accepted A07 signal plan

NEXT VALID TRADING SESSION

live PSX session data
-> valid session opening prices
-> current manual account state
-> executable A07 order ticket
-> watcher import
-> existing C15 live state machine
-> durable notifications

The normal operator workflow after C17 should not require manually running
individual backfill, adjustment, ML, ticket-generation, or import commands.

---

# 2. Frozen C16 Baseline

C17 must preserve all behavior accepted under C16.

Accepted allocation:

A07_P4_25_P5_75

C17 must not modify:

- accepted P4 strategy behavior;
- accepted P5 strategy behavior;
- accepted allocation weights;
- frozen production model;
- accepted signal-plan semantics;
- accepted order-construction policy;
- C15 live-order state-machine semantics;
- watcher durable-event semantics.

Any required change to those behaviors is outside C17 and requires a new
contract.

---

# 3. Scope

C17 consists of four production stages.

## 3.1 EOD preparation

After an official PSX trading session has completed:

1. obtain authoritative PSX EOD OHLCV;
2. update daily_ohlc;
3. apply accepted adjustment logic;
4. verify required adjusted fields;
5. verify the intended signal date is present and complete;
6. fail closed if authoritative data is unavailable or inconsistent.

The existing full historical backfill of the watcher `indicators` table is
NOT part of the critical A07 production path unless a concrete dependency is
demonstrated.

## 3.2 Phase A — after-close ML production

Given a verified signal date:

daily_ohlc through signal_date
-> accepted live scorer
-> deterministic P4/P5 selections
-> accepted A07 signal plan

Phase A must NOT require execution-session open data.

Its output must be durable and sufficient for Phase B to execute later
without recomputing or changing the accepted signal decision.

## 3.3 Phase B — next-session open

On the intended execution date:

1. obtain opening data from the existing live PSX market-data path;
2. validate that data belongs to the intended execution session;
3. require sufficient valid opening data for the symbols required by the
   frozen Phase-A signal plan;
4. load operational account state;
5. call the accepted C16/C11 order-construction path;
6. emit canonical watcher-facing JSON;
7. import that exact JSON into stock-watcher;
8. verify the resulting watcher run.

Phase B must NOT depend on execution-date daily_ohlc being available after
market close. The live session data path is the production source for the
current execution-session open.

## 3.4 Operational supervision

The orchestrated workflow must produce explicit durable status for:

- success;
- waiting for authoritative EOD;
- waiting for execution-session opening data;
- already completed/idempotent rerun;
- stale input;
- missing input;
- date mismatch;
- model/config integrity failure;
- ticket conflict;
- watcher import failure.

Failures must be visible through the existing notification infrastructure
where practical.

Silent fallback to older data is prohibited.

---

# 4. Trading-Date Rules

C17 must distinguish:

- wall-clock date;
- signal trading date;
- execution trading date.

It must not assume that the next calendar weekday is necessarily a PSX
trading session.

The implementation must fail closed if it cannot establish the intended
execution trading date.

The accepted C16 historical pair:

signal_date    = 2026-08-13
execution_date = 2026-08-17

must remain a regression case because it crosses the 2026-08-14 market
holiday and weekend.

---

# 5. Live Open Contract

C17 must define one canonical representation of execution-session opening
prices.

Minimum logical fields:

- trade_date
- symbol
- open_adj or equivalent accepted execution-open field
- source/provenance

The implementation must determine from the existing watcher market-data
schema which live field is authoritative for session open.

It must NOT silently substitute:

- previous close;
- first arbitrary current-price observation;
- stale prior-session quote;
- synthetic open;

unless such behavior is explicitly defined and accepted by the C17 interface
contract.

Opening data must be validated against the intended execution date.

---

# 6. Phase-A Artifact Contract

Phase A must persist a frozen artifact describing the signal decision.

At minimum it must provide:

- allocation_id;
- signal_date;
- intended execution_date;
- selected symbols;
- target weights;
- accepted buy-limit/reference values that are frozen at signal time;
- provenance/hashes;
- model identity;
- code revision.

The Phase-B order ticket must trace back to this artifact.

Phase B must not silently regenerate a different signal decision.

---

# 7. Phase-B Ticket Contract

The watcher-facing artifact remains the C16 canonical JSON ticket.

Requirements:

- top-level JSON array;
- allocation_id = A07_P4_25_P5_75;
- one signal_date;
- one execution_date;
- execution_date > signal_date;
- schema directly accepted by stock-watcher;
- deterministic canonical serialization;
- durable SHA-256 identity;
- idempotent watcher import.

No envelope such as {"orders": [...]} is permitted.

---

# 8. Account State

Until a broker/account API is introduced in a later contract, production
account state remains an explicit operational input.

Expected source:

config/live_account.json

Logical schema:

{
  "cash_pkr": <non-negative number>,
  "positions": {
    "<SYMBOL>": <non-negative whole shares>
  }
}

C17 must fail closed if required account state is missing or invalid.

C17 must not hard-code PKR 50,000 or any other capital amount into production
logic.

Acceptance fixtures may use deterministic synthetic account state.

---

# 9. Idempotency

Every production stage must be safe to rerun.

Examples:

- rerunning EOD update must not corrupt daily_ohlc;
- rerunning Phase A for the same immutable inputs must produce the same
  decision artifact;
- rerunning Phase B with the same inputs must produce the same logical ticket;
- importing the same watcher ticket must not duplicate the run;
- repeating an already-processed live board condition must not duplicate
  durable transition events.

A conflicting artifact for the same allocation/signal/execution identity
must fail explicitly.

---

# 10. Failure Policy

Production must fail closed when any required condition cannot be proven.

Forbidden behavior includes:

- use latest available date when exact date was requested;
- silently reuse yesterday's Phase-A artifact;
- silently reuse stale open data;
- silently switch model/configuration;
- silently change execution date;
- silently regenerate signals using different inputs;
- silently overwrite a conflicting watcher ticket.

---

# 11. Notifications

C17 should use the existing watcher notification infrastructure rather than
introducing an unrelated notification system.

At minimum, operationally meaningful notifications should distinguish:

- Phase A completed;
- Phase B ticket generated/imported;
- production workflow blocked/failed.

Notifications are informational only.

They do not constitute broker execution.

---

# 12. Scheduling / Runtime

C17 may introduce scheduler/orchestrator code, service definitions, or
container integration as required.

The design must keep the individual stages callable independently for:

- testing;
- recovery;
- acceptance;
- manual troubleshooting.

A scheduler must invoke deterministic stage commands rather than embedding
un-testable business logic only inside timing code.

---

# 13. Explicitly Out of Scope

C17 does NOT include:

- StockIntel integration;
- StockIntel contract/API work;
- broker API integration;
- broker cash/position querying;
- broker order submission;
- autonomous trading;
- model retraining;
- new strategy research;
- P4/P5 optimization;
- allocation changes;
- signal-viewer changes;
- redesign of C15;
- general optimization of historical indicator backfill.

---

# 14. Repository Ownership

## psx-ml-research owns

- Phase-A scoring and selection;
- frozen signal-plan artifact;
- Phase-B accepted order construction;
- ML-side deterministic manifests/provenance.

## stock-watcher owns

- authoritative EOD ingestion;
- adjustment invocation/integration;
- live PSX board/session data;
- execution-session opening-price acquisition;
- ticket import;
- C15 state machine;
- durable operational notifications.

## psx-system owns

- this MASTER contract;
- cross-repository interfaces;
- system acceptance;
- compatible component revision pins.

Component agents must not independently modify psx-system MASTER/interface
contracts.

---

# 15. Acceptance Principle

C17 is not accepted merely because individual commands work.

Acceptance requires demonstration of the lifecycle:

verified EOD signal session
-> frozen Phase-A decision
-> next valid execution session
-> live opening-data acquisition
-> Phase-B canonical ticket
-> unchanged watcher import
-> controlled C15 transition

with repeat execution proving idempotency and deliberate bad-input cases
proving fail-closed behavior.

