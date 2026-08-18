# C16 Interface Authority

C16 defines the production handoff between:

    psx-ml-research
            |
            | A07 production order ticket
            v
    stock-watcher

The authoritative semantic requirements are defined in:

    MASTER-CONTRACTS/C16-MASTER-CONTRACT.md

## Rule

The existing accepted order-ticket schema is to be preserved if it is already
sufficient and compatible.

Neither component implementation agent may independently change the
cross-repository ticket schema.

If inspection discovers an incompatibility, the agent must report:

1. current producer schema,
2. current consumer schema,
3. exact incompatibility,
4. smallest proposed resolution,

and stop that interface-changing portion of implementation.

The system architect will then freeze any necessary schema revision here.

## Required invariants

Regardless of serialization details, a production ticket must unambiguously
carry or imply:

- allocation identity
- signal provenance
- execution date
- symbol
- side/action
- production order parameters required by the accepted C15 importer/runtime

The accepted allocation identity is:

    A07_P4_25_P5_75

No manual editing is allowed between ML production and watcher import during
system acceptance.
