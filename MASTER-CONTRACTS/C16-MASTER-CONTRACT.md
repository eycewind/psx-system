# C16 MASTER CONTRACT
## Production A07 Signal-to-Live-Order Handoff

**Status:** ACTIVE  
**System repository:** `psx-system`  
**Component repositories:**
- `psx-ml-research`
- `stock-watcher`

**Baseline date:** 2026-08-18

---

# 1. Purpose

C16 closes the remaining production gap between the accepted A07 research/live
selection logic and the already implemented stock-watcher live order runtime.

The required end-to-end production flow is:

    current production market/EOD data
                  |
                  v
          psx-ml-research
                  |
          current P4/P5 selections
                  |
                  v
             signal plan
                  |
          next-session open data
                  |
                  v
             order ticket
                  |
          C16 handoff boundary
                  |
                  v
           stock-watcher
                  |
             ticket import
                  |
                  v
       A07 live order state machine
                  |
                  v
        durable order events
                  |
                  v
            notification layer

C15 implemented the watcher-side live order runtime. During live testing that
runtime correctly produced zero order events because no current execution-day
A07 order ticket had been imported.

C16 must eliminate that missing production handoff.

---

# 2. Frozen Starting Baseline

## psx-ml-research

Commit:

    e549e6c462e922c260e8865c0b4c5d666750070b

Tag:

    pre-c16-ml-baseline

C16 implementation branch:

    c16-production-ticket-pipeline

## stock-watcher

Commit:

    9ef2382bb3a62c115d5f69d01985cab5fb3a51e1

Tag:

    pre-c16-watcher-baseline

C16 implementation branch:

    c16-production-ticket-handoff

The accepted baseline must not be rewritten.

---

# 3. Authority and Ownership

This MASTER CONTRACT is the cross-repository authority.

Component implementation agents:

- MAY modify only their own component repository.
- MUST comply with this MASTER CONTRACT.
- MUST NOT modify files in `psx-system`.
- MUST NOT redefine cross-repository interfaces independently.
- MUST report an interface blocker if compliance requires a system-level
  contract change.

Changes to this MASTER CONTRACT are owned by the system architect.

---

# 4. Scope

C16 consists of two component workstreams.

## C16-ML — psx-ml-research

Implement a production-safe method of generating current A07 P4/P5 selections
using the already accepted/frozen methodology, then feed those selections into
the existing live signal-plan and order-ticket construction path.

The existing accepted strategy/policy must remain frozen.

## C16-WATCHER — stock-watcher

Complete the production handoff from an ML-generated A07 order ticket into the
existing C15 live order runtime.

The existing C15 state machine, persistence semantics, and durable event
semantics are presumed accepted and must not be redesigned without a concrete
defect.

---

# 5. Explicitly Out of Scope

C16 MUST NOT:

1. Change A07 strategy parameters.
2. Re-run strategy research in order to choose new parameters.
3. Refit or replace accepted models merely because C16 is being implemented.
4. Change P4/P5 allocation policy.
5. Change `A07_P4_25_P5_75` without an explicit MASTER CONTRACT revision.
6. Add StockIntel or other broker API order placement.
7. Place real brokerage orders.
8. Implement autonomous capital allocation beyond the accepted A07 policy.
9. Redesign the C15 live-order state machine without demonstrated necessity.
10. Invent missing historical C13/C14/C15 contract documents as part of C16.

Broker execution/API integration is a later system contract.

---

# 6. Core Safety Principle: Fail Closed

Production signal/order generation must fail closed.

A production artifact MUST NOT be emitted as actionable if required inputs are:

- missing,
- stale,
- internally inconsistent,
- for the wrong trading date,
- outside the accepted universe/policy,
- from an unrecognized schema,
- or impossible to provenance.

No silent fallback to older data is allowed.

No "best available date" behavior is allowed for production mode.

Diagnostic/research mode may behave differently only if clearly separated from
production mode.

---

# 7. Date Semantics

C16 must distinguish at minimum:

## signal_date

Trading session whose completed information is used to create the signal.

## execution_date

Trading session on which the resulting order ticket may become actionable.

For the normal daily strategy:

    execution_date > signal_date

and the order ticket must explicitly identify its execution date.

An order for one execution date must never become eligible merely because the
watcher is running on another date.

All date handling must be explicit and deterministic.

---

# 8. C16-ML Requirements

## ML-R1 — Production P4/P5 selection generation

`psx-ml-research` must provide a production entry point that produces the
current P4 and P5 selections from the accepted frozen methodology.

The implementation must reuse the accepted feature/model/policy logic rather
than creating a parallel approximation.

It must support an explicit `as_of` or equivalent production date.

It must reject stale/missing required data.

## ML-R2 — No look-ahead

Selection generation may use only information legitimately available for the
requested production signal date.

Future observations must not influence production selections.

## ML-R3 — Determinism

Given identical:

- code revision,
- model/policy revision,
- production data,
- signal date,

the P4/P5 selection output must be identical.

## ML-R4 — Existing live adapter integration

Generated P4/P5 selections must feed the accepted existing live workflow rather
than requiring a manually prepared `LIVE_p4_p5_selections` file.

The accepted workflow currently includes the concepts of:

    signal
    open

and production construction of:

    signal plan
    order ticket

C16 may extend/wrap these entry points but should not duplicate their policy
logic.

## ML-R5 — Signal-plan production

For an eligible signal date, the production path must generate a signal plan
using current generated selections and the required signal-date market data.

## ML-R6 — Order-ticket production

Given:

- accepted signal plan,
- valid execution-session open data,
- valid account state,
- execution date,

the production path must generate the A07 order ticket using the accepted
production order-construction logic.

## ML-R7 — Provenance

Every production run must leave sufficient provenance to determine at minimum:

- code revision,
- signal date,
- execution date where applicable,
- allocation ID,
- data/input identity,
- selection output identity,
- signal-plan identity,
- order-ticket identity.

A cryptographic file hash is preferred for final handoff artifacts.

## ML-R8 — Production artifacts

Production artifacts belong beneath the canonical:

    artifacts/live/

hierarchy.

Existing accepted artifact naming/layout should be preserved where practical.

C16 must not create a competing arbitrary artifact hierarchy.

---

# 9. C16-WATCHER Requirements

## W-R1 — Preserve C15

The following accepted C15 capabilities must remain operational:

- A07 ticket import
- idempotent import
- dynamic A07 recorder universe
- persistent live order state machine
- durable order events
- durable notification delivery tracking

## W-R2 — Consume the production order ticket

The watcher must consume the A07 production order ticket generated by the ML
component without manual schema editing.

The C16 boundary is therefore:

    psx-ml-research order ticket
                ->
    stock-watcher ticket importer

## W-R3 — Schema compatibility

The watcher must validate the incoming ticket before making it actionable.

Unknown or incompatible schemas must fail closed.

## W-R4 — Allocation identity

Production tickets must correspond to the accepted allocation:

    A07_P4_25_P5_75

unless the MASTER CONTRACT is revised.

## W-R5 — Execution-date enforcement

Only orders whose execution date matches the active trading date may enter
live processing.

Old tickets must not silently become current orders.

Future tickets must not become current orders early.

## W-R6 — Idempotent handoff

Re-importing the same production ticket must not create duplicate logical
orders.

## W-R7 — Provenance retention

Watcher persistence must retain enough information to identify the imported
production ticket/run and trace a live order back to its source ticket.

## W-R8 — Runtime processing

Once a valid current ticket is imported, the existing live market board
processing must be able to transition eligible A07 orders according to C15
rules.

## W-R9 — Durable event behavior

A qualifying state transition must create the expected durable event exactly
once.

Repeated processing of unchanged/equivalent board state must not create
duplicate logical events.

---

# 10. Cross-Repository Interface

The canonical cross-repository business artifact is the A07 production
**order ticket**.

C16 must preserve the existing accepted order-ticket schema wherever possible.

The implementation must not independently invent different ML and watcher
versions of the ticket schema.

If the current schemas are already identical and sufficient, C16 freezes that
schema.

If a concrete incompatibility exists, both component agents must report the
blocker before changing the cross-repository schema.

The system architect will then revise:

    interfaces/C16/

before component implementation continues.

---

# 11. Handoff Manifest

C16 should provide, either directly or through retained run metadata, a
machine-readable handoff identity containing at least:

    contract_id
    allocation_id
    signal_date
    execution_date
    ticket identity/hash
    producer code revision

Exact serialization is subordinate to preserving compatibility with the
existing accepted runtime.

The manifest is provenance, not a replacement for the production order ticket.

---

# 12. No Shared Database Requirement

C16 does not require the two repositories to mutate one shared database.

The order ticket is the explicit system boundary.

Either component may read production data from configured sources, but
cross-repository behavior must remain reproducible from explicit artifacts and
provenance.

---

# 13. Acceptance Requirements

C16 is not accepted merely because unit tests pass independently.

It requires cross-repository acceptance.

## SYS-AT01 — Frozen baseline

Both component C16 branches descend from the pre-C16 tagged baselines.

## SYS-AT02 — Current selection generation

ML production mode can generate P4/P5 selections for a supported signal date
without a manually authored live-selection file.

## SYS-AT03 — Deterministic selection

Repeating ML production selection generation with identical inputs produces
identical business output.

## SYS-AT04 — Stale-data rejection

Intentionally stale/missing production inputs fail closed and produce no
actionable production order ticket.

## SYS-AT05 — Signal plan

Generated current selections successfully feed the accepted signal-plan path.

## SYS-AT06 — Order ticket

A valid signal plan plus valid next-session open/account inputs produces an A07
production order ticket.

## SYS-AT07 — Direct watcher compatibility

The unmodified ML-produced order ticket is accepted by the watcher importer.

No hand editing of columns or values is permitted.

## SYS-AT08 — Idempotent import

Importing the same ticket twice creates no duplicate logical orders.

## SYS-AT09 — Execution-date gate

A ticket for a different execution date does not become eligible for live
processing.

## SYS-AT10 — Live state transition

With a controlled board fixture satisfying an accepted C15 trigger condition,
an imported current order transitions as expected.

## SYS-AT11 — Durable event exactly once

The transition in SYS-AT10 creates the expected durable event exactly once.

Repeated equivalent processing produces no duplicate logical event.

## SYS-AT12 — Provenance

The resulting watcher order/event can be traced back to the ML production
ticket and production run.

## SYS-AT13 — Regression

Existing accepted component tests continue to pass.

## SYS-AT14 — No broker execution

No acceptance test or runtime path created by C16 places a real brokerage
order.

## SYS-AT15 — Policy freeze

The implementation changes no accepted A07 strategy/model/allocation policy.

---

# 14. Required Component Deliverables

Each component repository must create:

    contracts/C16-CONTRACT.md
    contracts/C16-DELIVERY.md

The component contract must reference C16 MASTER CONTRACT authority and map its
implementation tasks to the applicable MASTER requirements.

The delivery report must include:

- implementation summary,
- files changed,
- commands/tests actually executed,
- actual test results,
- limitations,
- any unresolved blocker,
- final commit hash.

Delivery reports must contain real evidence, not planned or fabricated output.

---

# 15. Implementation Discipline

Each Codex implementation session must:

1. Inspect existing implementation before modifying it.
2. Prefer reuse over parallel implementation.
3. Preserve accepted policy behavior.
4. Add tests before claiming completion.
5. Never change the MASTER CONTRACT.
6. Stop and report if cross-repository interface requirements conflict.
7. Commit C16 independently on the dedicated C16 branch.
8. Do not merge into `main`.

---

# 16. Merge Authority

Component implementation completion is not system acceptance.

After both component deliveries:

1. system-level acceptance will be run,
2. component commits will be pinned in `psx-system`,
3. acceptance evidence will be reviewed,
4. only then may C16 be accepted and component branches merged.

