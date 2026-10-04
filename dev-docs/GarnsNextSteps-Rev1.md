# Garns — next steps

**Rev 1 · 2026-09-05 · workspace `garns-wz`**

> **Superseded 2026-10-04.** The Garns v9-6 product tree, now in `garns/`, carries
> the accepted decisions (ADRs A1–A16) and the W0–W8 work-package plan that replace
> this document. The move itself is recorded in
> [GarnsV9-6-MigrationPlan-Rev1.md](GarnsV9-6-MigrationPlan-Rev1.md). This file is
> kept as a historical record; do not plan from it.

A phased plan for the work that is independent of Glade. Glade is in design; the
`garns-glade` adapter shape sketched in `garns/INTEGRATION.md:353-424` and every
Taut-facing concern are **deliberately out of scope here** and are not planned,
sized or sequenced below. Nothing in this plan blocks on, or is blocked by, that
design.

---

## 1. Where the tree actually is

Measured on this checkout today, not cited from the cut manifest.

| Member | Size | Verification run today | Result |
|---|---|---|---|
| `garns` (Python) | 8,017 LOC across 22 modules, ~350 KB of documentation in 11 files | `unittest discover -s tests -t .` | `Ran 103 tests … OK` |
| `garns` gates | 128 checks over G0–G11 | read from committed `gate-report.json` (not re-run — `check.py` rewrites it in place) | `all_pass: true` |
| `garns-rust` | **133 LOC total**, 2 crates, 4 tests | `cargo test --workspace`, `cargo fmt --check` | pass, pass |

Both members are clean, on `main`, one `Initialize Garns workspace from ratified
v9-5` commit each, with GitHub origins registered (`owebeeone/garns`,
`owebeeone/garns-rust`) and nothing pushed.

**The asymmetry is the headline.** `garns` is a ratified, gate-covered,
mutation-tested compiler with a real corpus including an `EVERBILITY` world.
`garns-rust` is 133 lines that recognise four string constants and an enum of
seven stage names. It does not decode a single artifact.

```
garns-artifact   58 LOC   4 contract-id consts + Contract::recognize -> Option<Self>
garns-runtime    75 LOC   Stage enum (7 variants) + a Refusal struct with public fields
```

That is the correct place to have stopped — `garns-rust/AGENTS.md` says reject
unknown identifiers rather than guess — but it means **every Rust item below is
greenfield**, not extension.

---

## 2. The chokepoint: three unversioned contracts

`garns/contracts/contract-ids.json` versions three artifacts and lists three as
`pending`:

```json
"contracts": { "storage_binding", "generated_manifest", "generated_index" },
"pending":   [ "typed_ir", "diagnostic_envelope", "result_encoding" ]
```

Verified: `generated/SALES/manifest.json` carries `"schema":
"garns-v9-5/generated-manifest/1"`; `generated/SALES/ir.json` carries **no
`schema` key at all** — its top-level keys are `program`, `scope_paths`,
`storage`, `world`. The 262-code diagnostic catalogue exists only as 76 KB of
prose in `DIAGNOSTICS.md`; there is no machine-readable code list anywhere in
`src/`, `tools/` or `corpus/`.

Those three pending contracts are consumed by **every** Rust work item and by
the PostgreSQL parity gate. `ARCHITECTURE.md:24-32` already names this ordering
and it is right. The plan below adopts it as Phase 0 and refuses to start
Phase 2 or Phase 3 before it lands.

The alternative — decoding Python dataclasses from the other side — is
explicitly forbidden by `contracts/README.md` and `garns-rust/AGENTS.md`, and
would be the single most expensive mistake available here.

---

## 3. Phases

Phases are milestones. Steps are single goals with an **aspirational < 500 LOC**
budget. `[par]` marks steps that can be taken by an independent agent without
waiting on a sibling in the same phase.

### Phase 0 — Version the portable contracts *(foundational; blocks P2 and P3)*

The refusals and digests make this phase unusually sharp: get the emission right
once, regenerate once, and everything downstream has a fixture to test against.

| Step | Goal | Notes |
|---|---|---|
| **P0.1** | Give the typed IR a schema identifier and emit it | Add `"schema": "garns-v9-5/typed-ir/1"` to `ir.json`; register it in `contract-ids.json`. **This changes bytes**, so `ir_digest`, every `manifest.json`, every `tree_sha256` in `generated/INDEX.json`, and G9's byte-for-byte reproduction all move together. Do it as one atomic regeneration via `tools/generate_all.py` (which deletes its output tree first — expected). |
| **P0.2** [par] | Publish a machine schema for the typed IR | JSON Schema (or equivalent) covering the 18 `program` collections. The canonical encoder in `ir.py` is the source of truth; the schema must be *derived from* or *checked against* it, not hand-transcribed, or it rots on the next IR node. Add a gate check that every `$`-tagged dataclass appears. |
| **P0.3** [par] | Version the diagnostic envelope | `(code, stage, file, line, column, detail)` as `garns-v9-5/diagnostic/1`. Emit the 262-code catalogue as a **generated** `diagnostics.json` extracted from `src/` raise sites, and make `check_docs.py` assert `DIAGNOSTICS.md` and the JSON agree. Today the prose catalogue is the only registry and nothing enforces it. |
| **P0.4** | Decide the fate of the `generate` stage | `refuse.py:11-19` declares seven stages; `generate` has **zero** raise sites in `src/`. Either keep it in the versioned envelope with that documented, or drop it to six. Freezing an envelope with a dead variant is the sort of thing a second runtime inherits forever. Cheap now, permanent later. |
| **P0.5** | Version the result and parameter encodings | `garns-v9-5/result/1`: rows, the hidden `$k`-prefixed key columns, optional `total`. Plus the parameter contract: named binding, `list_of` givens JSON-encoded for `json_each` (`engine.py:305-332`), and the `[type_code, text]` cell encoding the parity runner prints. `Plan` (`lower_sqlite.py:56-70`) already implies all of this; nothing states it. |
| **P0.6** | Commit a cross-language conformance fixture set | Golden JSON in `garns/corpus/contracts/` that `garns-rust` tests read verbatim. Per `ARCHITECTURE.md:20-22` this is the **only** legal cross-language coupling — versioned envelopes and golden fixtures, never a source import. Without this step Phase 2 has nothing to assert against. |

**Exit:** `contract-ids.json` has an empty `pending` list; a Rust test can decode
a committed fixture and refuse an unknown version.

### Phase 1 — Close the declared-but-unconsumed gaps *(independent; ships value immediately)*

`LIMITATIONS.md:65-88` calls these "the cheapest gaps to close … and the most
dangerous to assume closed, because nothing refuses when you rely on them."
This phase does not depend on Phase 0 and can run fully in parallel with it.

Each step has the same three-way decision, and **the third option is the one to
refuse**: consume the fact, refuse the fact, or leave it silently accepted.

| Step | Goal | Notes |
|---|---|---|
| **P1.1** | Make `generated <targets>` gate emission | Verified defect: `generate.py:55-97` never reads `world.world.generated`; a world declaring `generated python` still gets the full `rust/` tree, `main.rs` and `sqlite.rs`. Every corpus world declares both targets, so the committed evidence *cannot* show this — add a single-target world to the corpus in the same step. |
| **P1.2** [par] | Consume or refuse the deployment items | `at`, `mode`, `snapshot`, `pool` and a world's `durability` are required, type-checked, carried into `ir.Deployment`, and read by nothing. `Store(world, path)` takes its path from the caller. Only `deployment.ship` is consumed (`evolution.py:339`). Recommend: refuse for now, consume in Phase 3 when a second engine makes `pool` and `mode` mean something. |
| **P1.3** [par] | Indexes, and the flags that should drive them | `CREATE INDEX` appears **zero** times in `src/` and in every committed `schema.sql` (verified). The `filter` flag on uses and links, and `order`, are the natural hints and are themselves unconsumed. These are one work item, not two. |
| **P1.4** [par] | Decide on `ordered_within`, `history kept`, `pattern`, `length` | Same three-way decision. `dimension` is checked only for positivity on a `Vector` intent. |
| **P1.5** [par] | Hygiene: `tools/check.py --quick` is parsed and ignored | `check.py:688,702,622` — `g11` never reads it, so every gate runs at full cost regardless. Either implement or delete the flag. |
| **P1.6** [par] | Fix the R1 P3 label defect | The check and test named *"engine inherited through `extends`"* actually exercise an **override** in an extending deployment (`tools/check.py:377-379`, `tests/test_repair_engine_lowering.py:65-69`). Rename, and add the missing case where the *base* itself carries the unlowered engine. Do this before touching engine code in Phase 3, not after. |

### Phase 2 — Rust API *(depends on Phase 0)*

Greenfield. `garns-rust`'s workspace lints set `unsafe_code = "forbid"`, which
means the 105-line raw `#[link(name = "sqlite3")]` FFI parity instrument
(`src/garns_rust/sqlite.rs`) is **off the table by policy** as a foundation —
consistent with `garns-rust/AGENTS.md`, which says so explicitly. A driver crate
is required, and choosing it is a decision, not a detail (see §5).

| Step | Goal | Notes |
|---|---|---|
| **P2.1** | `garns-artifact`: decode, don't just recognise | Today `Contract::recognize` returns an enum and stops. Add serde types for the storage binding, generated manifest and generated index, decoding the P0 fixtures. Unknown version ⇒ typed error, never a guess. |
| **P2.2** [par] | `garns-artifact`: typed IR decode | Against P0.1/P0.2. Largest single Rust step; likely needs splitting by IR collection to stay near budget. |
| **P2.3** [par] | `garns-runtime`: the diagnostic envelope, round-tripped | Decode/encode `Refusal` against P0.3 and the generated `diagnostics.json`. The existing `Refusal` struct has public `String` fields and no constructor, no `Display`, no `std::error::Error` — that is a stub shape, not an API shape. |
| **P2.4** | `garns-runtime`: result decoding | Against P0.5: rows, hidden `$k` keys, optional total, and the `[type_code, text]` cell encoding. |
| **P2.5** | `garns-sqlite`: real execution | The first crate that touches a database. **It must implement what the generated Rust surface deliberately does not** — `surfaces.py:30-42` emits constants only; parameter validation, scope binding, capability checks, the live bound and nested-child attachment all live in Python `Engine` (`engine.py:266-332`). Porting the surface is not porting the engine. |
| **P2.6** | Turn the parity gate into a cargo test | G11 today shells out to `rustc -O` against a generated tree (`check.py:622-683`). Once P0.6 fixtures exist, parity belongs in `cargo test --workspace`, reading fixtures, with the rustc harness kept only as the evidence trail it was built to be. |

**Explicitly not in this phase:** any Rust surface for verbs (there is no verb
runtime to mirror — see P4.1), and any adapter for a host system.

### Phase 3 — PostgreSQL *(depends on Phase 0; P3.0 depends on P1.6)*

The refusal is the specification. `postgres` is a *recognised* engine
(`ENGINES`) that is not a *lowered* one (`LOWERED_ENGINES`), and a deployment
naming it refuses `ENGINE_LOWERING_ABSENT` at `validate`, before any effect
(`resolve.py:27-28`, `:1293-1299`). It disappears when a lowering exists and not
one step earlier.

| Step | Goal | Notes |
|---|---|---|
| **P3.0** | Build the dialect seam that does not exist | **Prerequisite refactor, and the riskiest step in this plan.** Eight import sites bind `lower_sqlite` directly: `generate.py:18`, `surfaces.py:5`, `evolution.py:16` and `:231`, `capture.py:18`, `cli.py:21`, `engine.py:18` and `:551`, plus `mutants.py:23`. Extract a dialect protocol plus a registry keyed by engine name. **Strictly behaviour-preserving**: 103 tests, 128 gate checks and 168 mutant outcomes must all be unchanged. A stray `if engine == "postgres"` outside the registry is exactly the schema-name branching G10 scans for (`check.py:584-600`). |
| **P3.1** | `lower_postgres.py` — reads | Same `Plan` / `ChildPlan` contract. `EXTENDING_GARNS.md:262-284` already inventories the SQLite assumptions to displace: `json_each` for `in` and `:_parents`, `instr` for `contains`, `CAST(… AS REAL)` for division, `ROW_NUMBER() OVER` for `rank` and `last N`, and the `calls.py:25-41` static-call templates. |
| **P3.2** [par] | `lower_postgres.py` — DDL | `INTEGER PRIMARY KEY` identity, storage classes (`types.py:106-114`, incl. `Instant → INTEGER` — Postgres is the moment to ask whether that is still the right answer), `CHECK`, `UNIQUE`, enforced `FOREIGN KEY`, family discriminator. |
| **P3.3** | The Postgres store and engine | Connection and transaction control (no `PRAGMA foreign_keys`, no `executescript`), `RETURNING` in place of `cur.lastrowid`, and **SQLSTATE in place of `"UNIQUE" in str(exc)`** for splitting `KEY_DUPLICATED` from `CONSTRAINT_VIOLATED` (`engine.py:484-490`, `:517-522`, `:535-540`) — that string test is the single most fragile SQLite assumption in the codebase. Parameter binding style. |
| **P3.4** [par] | Capture for Postgres | The SQLite mechanism is `AUTOINCREMENT` changelog sequences plus `CREATE TRIGGER … OLD/NEW` (`lower_sqlite.py:666-700`), with `PRAGMA table_info` as the coverage denominator (`capture.py:31-45`). Keep the *contract* (typed deltas, one revision) and replace the mechanism; `information_schema` replaces the pragma. Logical decoding is a later option, not the first one. |
| **P3.5** [par] | Migration for Postgres | `evolution.py:220-350` uses `PRAGMA table_info`, SQLite `ALTER TABLE … RENAME/ADD/DROP COLUMN`, an `ANY`-typed quarantine column and `sqlite3.OperationalError`. |
| **P3.6** | Parity, then and only then the registry entry | Postgres rows must equal SQLite rows over the shared corpus, canonically encoded per P0.5. Add mutants and refusal fixtures for the new backend's failure modes. **Add `"postgres"` to `LOWERED_ENGINES` last.** `check.py::engine_lowering_regression` and `tests/test_repair_engine_lowering.py` are not deleted when this lands — they become the template for whichever engine is next recognised-but-unlowered. |

### Phase 4 — Python API depth *(independent of P0/P2/P3)*

This is the "Python API" half of the question, and it splits cleanly into one
large semantic gap and a set of production-readiness items.

| Step | Goal | Notes |
|---|---|---|
| **P4.1** | A verb runtime | The largest declared-but-unconsumed gap in the build. `alias`, `bulk`, `restricted` and `compound` resolve into IR and validate fully — derived-verb membership, bulkable verbs, set targets and types, accepted inputs, step ordering, bind types, **20 refusal codes** — and then *nothing executes them and no surface is generated*: `generate.py:72` iterates `world.reads` only. Callers write through `Engine.transaction` + `mint`/`change`/`delete` instead. Will exceed one step's budget; plan it as its own sub-milestone (execution, then generated surface, then corpus + mutants). |
| **P4.2** [par] | Concurrency and lifecycle: from script to service | One `sqlite3.connect(path, isolation_level=None)` (`engine.py:164-168`), no pool, no thread-safety statement, no `SQLITE_BUSY` retry, no async. `LiveEngine` has `subscribe` and **no `unsubscribe`**; `Registry` has no removal; every `Instance` retains its full materialised keyed state *and* an append-only log of every batch it ever emitted, neither trimmed (`live.py:65-88`, `:110-112`, `:151`). Fine for a gate run. This is the gate for anything that hosts Garns. |
| **P4.3** [par] | The three measured inefficiencies | `with_total` is a second `COUNT(*)` — two round trips per paged read (`lower_sqlite.py:591-596`); `Engine.row_values` selects *every* mapped column to build a before-image rather than projecting touched fields (`engine.py:266-279`); invariants are re-lowered from scratch and run as one `SELECT` per invariant per written row (`engine.py:544-564`). All correctness-neutral, all cheap to fix, none urgent until P4.2 makes throughput matter. |
| **P4.4** [par] | Live-instance partitioning coarseness | An unscoped instance registers under `global` **and** a `*` partition per footprint atom, so it is probed for writes in every scope (`live.py:194-200`). Cost, not correctness — but it is the one place where the "routing never iterates the registry" claim is weakest. |
| **P4.5** [par] | CLI beyond four verbs | `resolve`, `generate`, `ddl`, `execute`; nothing selected by position; a fresh store per invocation (`cli.py:71-88`). Enough for evidence, not for a development loop. Scope this only if someone is actually using the CLI to build a world. |
| **P4.6** [par] | Capabilities are caller-asserted | `Engine.execute(..., capabilities={...})` compares against a set the caller passes in; `writer` is a name check (`engine.py:328-329`, `:89-91`). There is no principal and no authorisation layer, by design. Flagged here so it is a **decision** rather than a discovery: if Garns is ever embedded behind an untrusted caller, this is the hole. |

### Phase 5 — Distribution *(independent, small, and gated on a decision)*

Both members build and test clean, neither is published, and `garns-wz` has one
commit. Given the standing OSS-distribution work elsewhere in the estate, this
should be a deliberate call rather than drift.

| Step | Goal | Notes |
|---|---|---|
| **P5.1** | Decide the publication posture | `garns` `0.1.0a1` wheel and sdist are already built in `dist/`; `garns-rust` is `0.1.0-alpha.1` and not on crates.io. Public, or private-for-now? Everything else in this phase follows from that answer. |
| **P5.2** | Stop shipping the parity instrument in the product wheel | `pyproject.toml` force-includes `src/garns_rust/sqlite.rs` into the wheel. That file is 105 lines of raw FFI whose only job is making Python and Rust row encodings comparable — `ARCHITECTURE.md` classifies it as conformance machinery. It should not travel to a `pip install garns` user without a stated reason. |
| **P5.3** [par] | CI that runs what the docs promise | Three commands (`check_docs.py`, the unit suite, `check.py`) plus `cargo test --workspace` and `cargo fmt --check`. Note `check.py` **rewrites `gate-report.json` in place** and does **not** run `tests/` — CI must run both readings and must not commit the rewritten report back. |

---

## 4. Sequencing

```
P0 Contracts ──┬────────────► P2 Rust API
               └────────────► P3 PostgreSQL (P3.0 seam also needs P1.6)

P1 Unconsumed gaps  ─── independent, start now, fully parallel with P0
P4 Python depth     ─── independent (P4.1 is its own sub-milestone)
P5 Distribution     ─── independent, gated on the P5.1 decision
```

**Recommended order of attack, and why:**

1. **P0 and P1 together, now.** P0 is the only thing standing between the
   current tree and any second-language or second-backend work, and it is
   cheap: identifiers, schemas and fixtures, not semantics. P1 needs no
   coordination with it and closes the gaps that silently accept input today.
2. **P3.0 next, alone, as a settled-tree refactor.** The dialect seam touches
   nine import sites across eight modules and must move zero test outcomes. It
   is the one step in this plan where doing it concurrently with other work will
   cost more than it saves.
3. **P2 and P3.1+ then fan out.** Once contracts are versioned and the seam
   exists, the Rust crates and the Postgres lowering are genuinely independent
   lanes with no shared files.
4. **P4.1 (verb runtime) whenever there is an owner.** It is the largest
   semantic gap and it touches `resolve`/`engine`/`generate`, so it wants a
   dedicated lane rather than a slot between other work.

---

## 5. Decisions needed before the work starts

These change what gets built, not just how.

1. **Rust SQLite driver.** `rusqlite` (thin, sync, bundled SQLite available) or
   `sqlx` (async, multi-backend, compile-time checked — but Garns generates its
   SQL, so compile-time checking buys little). The workspace forbids
   `unsafe_code`, so "port the FFI shim" is not a third option. This choice also
   sets whether `garns-sqlite` is sync or async, which propagates to every
   consumer.
2. **`Instant → INTEGER`.** `types.py:106-114` stores instants as integers with
   no timezone semantics anywhere. Phase 3 is the natural moment to revisit it,
   and revisiting it after Postgres ships is a migration.
3. **The `generate` stage variant** (P0.4): keep a stage with zero raise sites
   in the frozen envelope, or drop to six.
4. **Unconsumed facts** (P1.2/P1.4): consume, or refuse. Leaving them silently
   accepted is the option to rule out explicitly.
5. **Publication posture** (P5.1).

---

## 6. Not in this plan

* The `garns-glade` adapter shape (`INTEGRATION.md:353-424`), and everything
  downstream of it. Glade is in design; re-open this only against that design.
* Taut, and any host-system adapter.
* `token_bound`, reported by G11 verbatim as `"unverified: not measurable from
  inside the build"` (`check.py:625`). It is a review-lane concern, not a
  product one, and no measurement exists to extend.
* G12 comparative closure — the reviewers' gate, not the build's.
* Rewriting `tools/make_mutants.py` or `tools/migrate_seed_corpus.py`. Both
  delete their output trees first and belong to no gate; they are lane
  machinery, not product.
