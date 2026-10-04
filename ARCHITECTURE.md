# Workspace architecture

The workspace is a development composition, not a permission for circular
dependencies.

```text
garns  (the one product root, D8)
  Python compiler: source → qualified IR → WorldIR → Plan
       │
       │  published, versioned artifacts (A17) + shared conformance vectors
       │
       ├──► Python runtime  (src/garns/…, sdax)
       └──► Rust runtime    (crates/…, tokio + sdax-rs; planned, D9a)
                 │
                 ▼
external adapters such as garns-glade
  consume Garns and their host system
```

Rust is a first-tier product language alongside Python (operator decision D9,
2026-10-04). There is still exactly one compiler: Garns source resolves once, in
Python. The two runtimes meet the compiler only at versioned artifacts — the plan
encoding, result shapes, typed values, refusals, revision and ledger envelopes,
and ship and migration requests — and both are held to the same conformance
vectors. Neither side imports the other's source. That boundary, now drawn inside
one tree, replaces the old repository split.

Product direction and work packages live inside `garns`:
`garns/docs/GARNS_DIRECTION.md` (direction and invariants),
`garns/docs/PRODUCT_LAYOUT.md` (the W0–W8 packages and their path ownership),
`garns/docs/adr/` (decisions A1–A16) and
`garns/dev-docs/GarnsV9-6-RustFirstTierAmendment.md` (the Rust packages R1–R5 and
decisions D9a–D9d, as a draft for the lane's review).

The `garns-rust` member is superseded by D9a: first-tier Rust code goes in-tree
under `garns/crates/`. The member is kept as it is; its two starter crates may
seed that work. The raw SQLite FFI runner (`garns/src/garns_rust/sqlite.rs`)
remains conformance evidence until the Rust runtime provides real parity.
