# C17 Live Open Interface

Status: FROZEN FOR C17 IMPLEMENTATION

## Purpose

Defines the execution-session opening-price interface between stock-watcher
and ML Phase B.

C17 must NOT wait for execution-date EOD daily_ohlc.

The production source is the live PSX portal data already persisted by
stock-watcher in `market_quotes`.

## Source

SQLite table:

`market_quotes`

Required source fields:

- `poll_ts`
- `symbol`
- `open`
- `source`

Production source must be:

`source = psx_portal`

## Important observed portal behavior

Historical inspection of the 2026-08-17 live session showed transient changes
in the PSX portal `open` field during the first few minutes after market open.

Examples:

- many symbols changed `open` at approximately 09:32 PKT;
- most changed back at approximately 09:33 PKT;
- IMAGE and OCTOPUS settled later;
- after 09:40 PKT no observed symbol changed `open`;
- first open observed at/after 09:40 matched authoritative EOD open for all
  40 compared symbols.

Therefore C17 must NOT use the first observed market-open value immediately
after 09:30.

## Normal-session settling rule

For a normal PSX session:

`settle_not_before = 09:40:00 Asia/Karachi`

A symbol's execution open may become eligible only from observations at or
after this boundary.

The implementation SHOULD confirm the same valid open value in at least two
qualifying observations before declaring that symbol settled.

Given the current approximately 90-second watcher polling interval, normal
Phase-B readiness is therefore expected around 09:41-09:43 PKT.

## Canonical logical record

Each settled execution open must expose:

- `trade_date`
- `symbol`
- `open`
- `first_qualifying_poll_ts`
- `confirmed_poll_ts`
- `confirmation_count`
- `source`

Where:

- `trade_date` is derived from the execution-session `poll_ts`;
- `open` is the raw `market_quotes.open` value;
- `source` must identify the live PSX source.

Do not call this value `open_adj`.

It is a live raw execution price, not a corporate-action-adjusted historical
research price.

## Validity rules

A settled open is valid only if:

1. `trade_date == intended execution_date`;
2. `source == psx_portal`;
3. `open` is finite;
4. `open > 0`;
5. the observation is not before the configured settle boundary;
6. confirmation requirements are satisfied.

## Required-symbol rule

Phase B must determine the symbols required by the frozen Phase-A decision.

If any required symbol does not have a valid settled execution open, Phase B
must remain WAITING/BLOCKED.

It must not silently omit that symbol.

## Forbidden fallbacks

C17 must never silently substitute:

- first arbitrary post-09:30 open;
- `current`;
- previous close / LDCP;
- previous-session open;
- execution-date future EOD data;
- another symbol's timestamp/value;
- synthetic open.

## Idempotency

For the same execution date and identical settled source observations,
extraction must produce the same logical open set.

Once Phase B has consumed a frozen settled-open set, later portal observations
must not silently rewrite the already-issued ticket.

A deliberate regeneration must be explicit and conflict-checked.

## Historical acceptance case

Execution date:

`2026-08-17`

Observed validation:

- first at/after 09:40 matched authoritative EOD open: 40/40;
- mismatches: 0;
- symbols with changing open after 09:40: 0.

This case must remain a C17 regression fixture/test.
