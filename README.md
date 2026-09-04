# Garns workspace

GWZ-managed development workspace for the Garns language and its downstream
Rust libraries.

| Member | Owns | Must not depend on |
|---|---|---|
| `garns` | Grammar, Python compiler/runtime, portable contracts, corpus, generated evidence, product documentation | `garns-rust`, Glade, Taut |
| `garns-rust` | Rust consumers of versioned Garns artifacts | Python implementation details, Glade, Taut |

Glade and Taut adapters belong downstream, initially in `glade-wz`. They may
depend on published Garns contracts and Rust crates; this workspace does not
materialize either project.

Use GWZ for workspace operations:

```sh
gwz status
gwz diff --stat
gwz forall garns-rust -- cargo test --workspace
```

Do not edit `gwz.conf/` directly. See `AGENTS_GWZ.md`.
