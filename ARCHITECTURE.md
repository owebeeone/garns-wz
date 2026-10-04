# Workspace architecture

The workspace is a development composition, not a permission for circular
dependencies.

```text
garns
  owns DSL + grammar + compiler + backend contracts
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

Product direction and work packages live inside `garns`:
`garns/docs/GARNS_DIRECTION.md` (direction and invariants),
`garns/docs/PRODUCT_LAYOUT.md` (the W0–W8 packages and their path ownership) and
`garns/docs/adr/` (decisions A1–A16, whose acceptance records are in
`garns/dev-docs/`). The five-step stabilisation list this file used to carry
belonged to the v9-5 cut and is superseded by those documents.

`garns-rust` has no v9-6 work package yet; its scope is an open decision. The raw
SQLite FFI runner (`garns/src/garns_rust/sqlite.rs`) remains conformance evidence;
it is not the implementation of `garns-rust`.
