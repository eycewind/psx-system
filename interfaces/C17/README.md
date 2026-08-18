# C17 Interfaces

C17 freezes two new cross-repository interfaces:

1. `PHASE-A-DECISION.md`
   - durable after-close trading decision;
   - Phase B consumes it without recomputing signals.

2. `LIVE-OPEN.md`
   - settled execution-session opening prices sourced from stock-watcher
     `market_quotes`;
   - normal-session observations before 09:40 PKT are not eligible;
   - required symbols must have valid settled opens.

The existing C16 canonical watcher JSON ticket remains unchanged.

These interfaces are system-owned. Component agents must report an interface
conflict rather than independently changing their meaning.
