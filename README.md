# PSX System

System-level architecture, integration contracts, interface definitions, and
cross-repository acceptance for the PSX research and trading platform.

## Components

- `repos/psx-ml-research`
  - research and model-development system
  - accepted A07 strategy/policy
  - live P4/P5 selection and production-order generation

- `repos/stock-watcher`
  - market-data recording
  - live A07 order-state processing
  - durable execution events and notifications

The component repositories remain independently versioned Git repositories and
are included here as Git submodules.

## System-level ownership

- `MASTER-CONTRACTS/`
  Authoritative cross-repository contracts maintained by the system architect.

- `interfaces/`
  Frozen schemas, fixtures, and interoperability specifications.

- `acceptance/`
  Cross-repository/system acceptance tests and evidence.

- `scripts/`
  Integration, validation, and system-maintenance utilities.

MASTER-CONTRACT files are authoritative and read-only to component
implementation agents. Component implementations may report an interface
blocker but must not modify a MASTER-CONTRACT.
