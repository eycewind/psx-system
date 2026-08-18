# C17 Phase-A Decision Interface

Status: FROZEN FOR C17 IMPLEMENTATION

## Purpose

Phase A runs after the signal trading session has closed and authoritative
EOD data has been prepared.

Its output freezes the accepted trading decision before the next execution
session begins.

Phase B must consume this frozen decision rather than rerun signal selection.

## Inputs

Phase A requires:

- exact `signal_date`;
- explicit intended `execution_date`;
- authoritative daily_ohlc through signal_date;
- accepted frozen production model;
- accepted feature configuration;
- accepted point-in-time universe/reference data.

Phase A must NOT require execution-session open prices.

## Policy freeze

Allocation:

`A07_P4_25_P5_75`

C17 must preserve the accepted C16:

- P4 methodology;
- P5 methodology;
- P4/P5 allocation;
- model;
- scoring behavior;
- signal-plan construction behavior.

## Durable outputs

Phase A must persist a durable decision bundle containing at least:

- signal date;
- intended execution date;
- allocation ID;
- selections;
- signal plan;
- model identity/hash;
- relevant input identities/hashes;
- code revision;
- generation timestamp;
- output hashes.

The implementation may retain the existing C16 Parquet artifacts where
appropriate.

A manifest must provide a stable identity for the complete Phase-A decision.

## Immutability

Once a Phase-A decision has been accepted for a given:

- allocation_id;
- signal_date;
- execution_date;

Phase B must use that exact decision.

Phase B must not silently:

- rescore;
- regenerate selections;
- change P4/P5 membership;
- change target weights;
- change model/configuration;
- select a different signal date.

## Idempotency

Repeating Phase A with identical immutable inputs must produce the same logical
decision.

A materially different decision for the same allocation/signal/execution
identity must be treated as a conflict and must not silently overwrite the
existing accepted decision.

## Phase-B relationship

Phase B consumes:

frozen Phase-A decision
+ settled execution-session live opens
+ operational account state

and invokes the already accepted order-construction path.

Phase B then emits the unchanged C16 canonical watcher-facing JSON ticket.

## Provenance

The Phase-B production manifest and/or ticket handoff metadata must provide
a traceable relationship back to the frozen Phase-A decision identity/hash.

Watcher order/event provenance must therefore ultimately be traceable to:

watcher event
-> watcher order/run
-> Phase-B ticket
-> Phase-A decision
-> frozen model/input provenance
