# C17 Interfaces

C17 introduces two system-level interfaces that must be frozen before final
acceptance:

1. `PHASE-A-DECISION.md`
   - durable after-close ML signal decision consumed by next-session Phase B.

2. `LIVE-OPEN.md`
   - canonical execution-session opening-price representation supplied from
     stock-watcher live market data to ML Phase B.

The existing C16 canonical watcher JSON ticket remains unchanged.

Component implementations must not independently invent incompatible forms of
these interfaces. Any mismatch discovered during implementation must be
reported back to the system architect.
