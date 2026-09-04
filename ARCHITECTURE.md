# Workspace architecture

The workspace is a development composition, not a permission for circular
dependencies.

```text
garns
  owns DSL + grammar + portable artifact contracts
       │
       ▼
garns-rust
  consumes versioned contracts; provides Rust libraries
       │
       ▼
external adapters such as garns-glade
  consume Garns and their host system
```

The `garns` member remains the dependency leaf. A Rust test may use committed
contract fixtures emitted by Garns, but the Python build and tests may not need
the sibling Rust checkout. Cross-language compatibility is established through
versioned artifact envelopes and golden fixtures, never by importing source
from another member.

The first cross-language stabilization work is:

1. Give the typed IR an explicit schema identifier and machine schema.
2. Version the diagnostic envelope.
3. Version parameter, row, hidden-key, total, and child-result encodings.
4. Version transaction, ledger delta, live batch, and capture envelopes.
5. Only then implement the real SQLite Rust backend.

The v9-5 raw SQLite FFI runner remains conformance evidence in `garns`; it is
not the implementation of `garns-rust`.
