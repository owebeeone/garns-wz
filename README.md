# Garns workspace

GWZ-managed development workspace for the Garns language.

| Member | Owns | Must not depend on |
|---|---|---|
| `garns` | The single convergent Garns v9-6 product tree: grammar, Python compiler, backend contracts and their reference models, ADRs, corpus, generated evidence, documentation and lane records — and, under `crates/` (planned), the first-tier Rust runtime | `garns-rust`, Glade, Taut |
| `garns-rust` | Superseded: first-tier Rust lives in-tree under `garns/crates/` (D9a). Kept as it is; its two starter crates may seed that work | Python implementation details, Glade, Taut |

`garns` was imported byte-exact from `datascad/garns-v9-6` on 2026-10-04, and v9-6
development continues here. Its direction is PostgreSQL as the primary backend and
SQLite as a supported secondary, behind an async-only public database API in **two
first-tier languages: Python (3.11 or later) and Rust** (operator decision D9,
2026-10-04). One Python compiler feeds both runtimes through versioned artifacts,
and both runtimes use the sdax family for lifecycle orchestration. Work inside
`garns` follows `garns/AGENTS.md` and its review loop. Start with:

- `garns/README.md` and `garns/docs/GARNS_DIRECTION.md` — what Garns is and where it is going
- `garns/docs/PRODUCT_LAYOUT.md` — the W0–W8 work packages and their path ownership
- `garns/dev-docs/GarnsV9-6-RustFirstTierAmendment.md` — Rust first tier: decisions D9a–D9d and packages R1–R5 (a draft for the lane's review)
- `garns/dev-docs/GarnsV9-6-Relocation.md` — the move, and how to resume the parked W1A11 review

The previous v9-5 B2 product cut is tagged `v9-5-b2-cut` in all three repositories.
The migration plan and its execution log are in
[dev-docs/GarnsV9-6-MigrationPlan-Rev1.md](dev-docs/GarnsV9-6-MigrationPlan-Rev1.md).

Glade and Taut adapters belong downstream, initially in `glade-wz`. They may
depend on published Garns contracts and Rust crates; this workspace does not
materialize either project.

Use GWZ for workspace operations:

```sh
gwz status
gwz diff --stat
gwz log --full
```

Do not edit `gwz.conf/` directly; see `AGENTS_GWZ.md`. **Until the W1A11 review gate
closes, do not run `gwz init --update`**: the frozen review manifest pins
`AGENTS_GWZ.md` by hash.
