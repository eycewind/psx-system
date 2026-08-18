# Pre-C16 System Baseline

Status: ACCEPTED
Date: 2026-08-18

This document freezes the component revisions forming the starting point for
system-level Contract C16.

## Component Baselines

### psx-ml-research

Commit:

    e549e6c462e922c260e8865c0b4c5d666750070b

Description:

    Normalize contract and artifact layout

Baseline includes the previously accepted live deployment adapter and the
subsequent repository-layout normalization.

Git tag:

    pre-c16-ml-baseline


### stock-watcher

Commit:

    9ef2382bb3a62c115d5f69d01985cab5fb3a51e1

Description:

    Normalize contract and artifact layout

Baseline includes C15.1 through C15.5:

- removal of the legacy watchlist runtime
- idempotent A07 order-ticket import
- dynamic A07 recorder universe
- persistent A07 live order state machine
- durable A07 event notifications

and the subsequent repository-layout normalization.

Git tag:

    pre-c16-watcher-baseline


## System Repository

The psx-system superproject pins exactly the component commits listed above.

These revisions constitute the immutable starting baseline for C16.

C16 implementation work must be performed on new component branches created
from these baselines. Existing accepted implementation and historical contract
artifacts must not be rewritten as part of C16.
