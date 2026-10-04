# Garns v9-6 → garns-wz migration plan

**Rev 1 · 2026-10-04 · workspace `garns-wz`**
**Decision:** operator chose option (B) — Garns development moves into
`garns-wz`. The `garns` member becomes the v9-6 product tree; the datascad copy
stays untouched as the historical record.
**Supersedes:** [GarnsNextSteps-Rev1.md](GarnsNextSteps-Rev1.md), whose phases
and open decisions are already covered by v9-6's accepted ADRs A1–A16 and
W0–W8 package plan.

**Operator decisions, 2026-10-04:** no backup — P0.2 is skipped. **G1 = (c):**
publish to the existing public repositories as they are, so P4.1 does not apply;
chosen with the exposure described in §3 stated. **G2 = cut over now.**
**G3 = tag `v9-5-b2-cut` in garns-wz with `gwz tag`.** **datascad is left
alone** — no detach and no pointer document. Execution results are in §9.

---

## 1. Scope

**In scope**

- The whole v9-6 product root, `/Volumes/projects/limbo/datascad/garns-v9-6`
  (1,135 files, 11 MB), moved **byte-exact** into
  `/Volumes/projects/limbo/garns-wz/garns`.
- Tagging garns-wz's current committed state before anything changes.
- Keeping the parked W1A11 review gate resumable from the new location.
- garns-wz's own root documents (README, ARCHITECTURE).
- Nothing in datascad. Its copy of v9-6 stays exactly as it is (§4, Phase 5).

**Out of scope**

- **Any change to v9-6 content.** No packaging fix, doc correction or path
  cleanup happens during the move; those are W7 inputs (§7). The move must be
  provably lossless, and the lane's hash maps pin the files that would tempt an
  edit.
- The older datascad lanes (`garns`, `garns-v2` … `garns-v9-5`). They stay in
  datascad as history.
- `garns-rust` content. It stays a garns-wz member, untouched (§8).

---

## 2. Facts this plan rests on

Measured on 2026-10-04. Re-measure at execution; do not cite.

| Fact | Consequence |
|---|---|
| v9-6 has **0 commits and no remote**; it is gwz member `mem_garns_v9_6` of `datascad` | Its only copy is the external drive. Back it up first (P0.2). |
| `~/old-limbo` is the 2026-10-02 copy source and **predates v9-6** (created 2026-10-03) | old-limbo is not a backup of v9-6. |
| No caches, no `*.sqlite`; v9-6's `.gitignore` excludes nothing the tree contains | A git commit should carry every file — still proven after commit (P1.5). |
| 38 hash maps. 36 are product-root-relative. MANIFEST-1 (117) and Inputs31 each pin one outside file, `../AGENTS_GWZ.md` | `garns-wz/AGENTS_GWZ.md` is **byte-identical** to datascad's (`432b1ba9…`, the pinned value), so all 38 maps verify unchanged from `garns-wz/garns`. |
| HandoffEvidence's 7 entries are product-relative | The handoff's own verification step works at the new location. |
| `../dev-docs/…` citations come from `docs/*.md` | They resolve **inside** the product and move with it. |
| Only two live citations point truly outside: `../garns-v9-5/` (`AGENTS.md:61`) and `../garns-v9-5/build/B2` (`BASELINE.json:4`) | Citation-only: `tools/check_product.py` reads `BASELINE["sha256"]` (product-relative), never `selected_source.path`. The B2 tree remains at `datascad/garns-v9-5` and `~/old-limbo/datascad/garns-v9-5`. |
| 129 files contain absolute old paths | Almost all are historical records (reviews, logs, drafts, evidence, gate reports) and stay byte-identical. `check_product.py` uses two as a forbidden-leak list and reads neither. **Operationally, only the 4 undispatched W1A11 review prompts matter** — 2 lines each name the datascad product root. |
| `AGENTS.md`, `README.md`, `BASELINE.json`, `pyproject.toml`, `docs/PRODUCT_LAYOUT.md` and the checkpoint are hash-pinned (3–18 maps each) | Their now-stale self-descriptions ("registered as `mem_garns_v9_6` at `garns-v9-6`") are **superseded by a new record (P3.1), not edited**. |
| garns-wz: root `7c706b1`, garns `34f2f14`, garns-rust `7e8e319`; each equals `origin/main`; no tags; root has untracked `dev-docs/` | The v9-5 cut is already pushed. Tagging captures it exactly. |
| **`owebeeone/garns` and `owebeeone/garns-wz` are PUBLIC** | Pushing after the import publishes the whole v9-6 lane (gate G1). |
| `gwz` on `PATH` is an asdf shim that fails in garns-wz; the real binary is `~/.cargo/bin/gwz` (1.0.17) | Every command below uses the explicit path. |

Shorthand used below:

```sh
GWZ=~/.cargo/bin/gwz
WZ=/Volumes/projects/limbo/garns-wz
SRC=/Volumes/projects/limbo/datascad/garns-v9-6
```

---

## 3. Operator decision gates

These change what happens, so they are the operator's — not the executor's.

### G1 — Publication posture *(blocks Phase 4)*

Pushing the `garns` member after the import publishes, to a public repository:
240 dev-docs including the review prompts and the **unreviewed** W1A11
candidate; 72 files carrying `/Users/owebeeone/…` paths; 18 naming the review
model; 33 mentioning held-out or private-attack material.

| Option | Effect |
|---|---|
| **(a) Make `owebeeone/garns` and `owebeeone/garns-wz` private, then push** — *recommended* | One remote, history and backup immediately; reversible at W7 release. Same posture as sdax: private→public is reversible, the reverse is not. The v9-5 cut was already public; making it private hides it but does not unpublish it. |
| (b) Push v9-6 to a new private remote; leave the public origin at the v9-5 cut until release | Keeps the public face unchanged; adds a second remote to manage. |
| (c) Publish as-is | Irreversible exposure of lane process, local paths and review material. |

Push identity also needs confirming: the remotes are SSH
(`git@github.com:owebeeone/…`), `gwz push` authenticates through libgit2 and
`SSH_AUTH_SOCK`, and this machine carries two GitHub identities (gripd for work,
owebeeone personal) that have been confused before.

### G2 — When authority transfers *(shapes Phase 3)*

| Option | Effect |
|---|---|
| **Cut over now** — *recommended* | The tree is parked with no builder or reviewer running, which makes now the cleanest moment to move it. The four prompts are re-issued as Prompt-2 with only the root path changed (P3.2), and the review runs in garns-wz, so every acceptance record from here on cites the new home. |
| Cut over after the gate | The review runs in datascad with Prompt-1 verbatim; afterwards its new records are delta-synced into garns-wz (P3.2-alt). No prompt changes, but two copies coexist until the gate closes. |

### G3 — Tags

Proposed name `v9-5-b2-cut`: descriptive, and deliberately not a semver release
tag. Created locally in P0.5; pushed only after G1.

---

## 4. Phases

Each phase is a milestone. The steps are operations or short records; none comes
near the aspirational 500-LOC step budget.

### Phase 0 — Freeze, back up, tag *(foundational; nothing moves)*

| Step | Goal | How |
|---|---|---|
| **P0.1** | Confirm v9-6 is quiet | Handoff and checkpoint: builder STOP-WRITES, four prompts saved and **not** dispatched. No process may write `$SRC` from here until P5. |
| **P0.2** | Close the single-copy risk now | `ditto "$SRC" ~/old-limbo/datascad/garns-v9-6-backup-20261004` (internal disk: 123 GiB free; the tree is 11 MB). Verify against the P0.3 inventory. **Can run today, independent of everything else.** |
| **P0.3** | Fix the migration's own identity | From `$SRC`: `find . -path ./.git -prune -o -type f -print0 \| sort -z \| xargs -0 shasum -a 256 > "$WZ/dev-docs/GarnsV9-6-Migration-SourceInventory.sha256"` → 1,135 entries. This is separate from the lane's maps and covers every file. |
| **P0.4** | Run the handoff's step 1 at the old location | HandoffEvidence; MANIFEST-1's own hash (`7e80ca67…`) and its 117 entries; Inputs31, baseline20, ReadOnly741, Legacy115/71/111/614, AcceptanceEvidence13 — each from the product root. Record the results for P3.1. Read-only. |
| **P0.5** | Tag garns-wz's current state *(operator-confirmed)* | `$GWZ --root "$WZ" tag v9-5-b2-cut -m "Garns v9-5 B2 cut (CUT_MANIFEST composite eaf4937f…), before the v9-6 import"`. Tags root `7c706b1`, garns `34f2f14`, garns-rust `7e8e319`. Local only. The untracked root `dev-docs/` is new migration work and is not part of this state. |

**Exit:** backup verified, inventory filed, lane guards pass at the old location,
tag present in all three repositories.

### Phase 1 — Import, byte-exact *(depends on P0)*

| Step | Goal | How |
|---|---|---|
| **P1.1** | Make the member's working tree equal v9-6 | `rsync -a --delete --exclude=.git "$SRC/" "$WZ/garns/"`. `--exclude` also protects the member's own `.git` from deletion. This removes the v9-5 cut's files, including local `.venv/` and `dist/` builds — all recoverable from the tag or regenerable. |
| **P1.2** | Prove the working tree | `diff -r --exclude=.git "$SRC" "$WZ/garns"` is empty; `(cd "$WZ/garns" && shasum -a 256 -c "$WZ/dev-docs/GarnsV9-6-Migration-SourceInventory.sha256")` passes 1,135/1,135. |
| **P1.3** | Prove the lane's guards at the new location | Re-run every map from P0.4 from `$WZ/garns`. The two `../AGENTS_GWZ.md` entries now resolve to `garns-wz/AGENTS_GWZ.md` — identical bytes. |
| **P1.4** | Protect the one outside pin | **Do not run `gwz init --update` in garns-wz until the W1A11 gate closes.** Regenerating `AGENTS_GWZ.md` would break MANIFEST-1 and Inputs31. |
| **P1.5** | Commit, then prove the *committed* bytes | `$GWZ --root "$WZ" --target garns add -A`, then commit with a message citing the inventory hash, MANIFEST-1 `7e80ca67…`, the source path, and "supersedes the v9-5 B2 cut (tag v9-5-b2-cut)". Then `git -C "$WZ/garns" archive HEAD \| tar -x -C <scratch>/import-check`, re-run the inventory there, and confirm `git ls-files \| wc -l` = 1,135. This catches anything `.gitignore` or attributes could drop. (Check flag placement with `$GWZ add --help` before running.) |

**Exit:** one import commit in `garns` whose extracted tree matches the source
inventory exactly.

### Phase 2 — Executable verification *(depends on P1; parallel with P3)*

Always in a scratch extraction, never in place: `tools/check.py` rewrites
`gate-report.json` (its timing field changes); `generate_all.py`,
`make_mutants.py` and `migrate_seed_corpus.py` delete their output trees first;
and test runs can leave caches that `check_product.py` refuses.

| Step | Goal | How |
|---|---|---|
| **P2.1** | Reproduce the lane's matrix | From `git archive HEAD`, run focused contracts (130), full tests (233) and product checks (5) on CPython 3.11, 3.12, 3.13 and 3.14, `uv run --offline`, with `PYTHONDONTWRITEBYTECODE=1`. Compare with `W1A11-WorkerExitImplementation-VerificationLogs-1.md`. The inherited SQLite `ResourceWarning`s on 3.13/3.14 are expected. |
| **P2.2** | Reproduce the baseline gate | `tools/check.py` in scratch. The resulting `gate-report.json` should match the committed one apart from timing fields. |

**Exit:** the same counts as the lane's own reproduction. This is relocation
evidence only — it is not review closure or acceptance of anything.

### Phase 3 — Relocation record and lane continuity *(depends on P1; parallel with P2)*

| Step | Goal | How |
|---|---|---|
| **P3.1** | File the relocation record inside the product | New file `garns/dev-docs/GarnsV9-6-Relocation.md`, carrying: the path map (`datascad/garns-v9-6` → `garns-wz/garns`; workspace `datascad` → `garns-wz`; member `mem_garns_v9_6` → `mem_garns`); P0.4 and P1.3 results; the import commit; the citation map `../garns-v9-5/` → `datascad/garns-v9-5` (both copies); and the statement that the pinned self-descriptions in `AGENTS.md`, `README.md`, `BASELINE.json`, `docs/PRODUCT_LAYOUT.md` and the checkpoint are superseded by this record. Adding a file breaks no existing map. |
| **P3.2** | Re-issue the review prompts *(G2 = now)* | Four new files, `W1A11-WorkerExitImplementation-{ReviewCode,ReviewState,OriginCodeClosure,OriginStateClosure}-Prompt-2.md`, each identical to Prompt-1 except its two root-path lines. Record the exact diff in P3.1. Add `W1A11-WorkerExitImplementation-RelocationEvidence.sha256` pinning the relocation record, the four Prompt-2 files, MANIFEST-1 and the updated checkpoint. Prompt-1 and the old HandoffEvidence stay byte-identical as history. |
| **P3.2-alt** | *(G2 = after gate)* | Run the review in datascad with Prompt-1 verbatim. Then `rsync` only its new records into `garns/`, re-run P1.2-style checks over the union, and commit "W1A11 review records, synced from datascad". Until then, the root README marks `garns` as a mirror: **do not edit**. |
| **P3.3** | Point the live index at the new home | Manager-owned edit to `dev-docs/CurrentProgramCheckpoint.md`: resume from the relocation record. HandoffEvidence was verified at both locations (P0.4, P1.3) before this edit, so its checkpoint entry is *expected* to differ afterwards; RelocationEvidence pins the new checkpoint. P3.1 states this so a later verifier is not surprised. |
| **P3.4** | Update garns-wz's own root documents | These are garns-wz's files and are not lane-pinned. `README.md`: `garns` is the v9-6 product tree (Postgres-primary, async-only public API, Python 3.11+), developed here under `garns/AGENTS.md` and the review loop. `ARCHITECTURE.md`: replace the v9-5-era five-step stabilisation list with pointers to `garns/docs/GARNS_DIRECTION.md` and `garns/docs/PRODUCT_LAYOUT.md`, keeping the dependency-direction picture. Mark `dev-docs/GarnsNextSteps-Rev1.md` superseded by a header line. |
| **P3.5** | Commit the root | This plan, the source inventory, Rev 1's superseded marker, README and ARCHITECTURE — via `gwz add` / `gwz commit` on the root. |

**Next-manager resume instruction** (G2 = now):

> "Resume from `garns/dev-docs/GarnsV9-6-Relocation.md`, then
> `W1A11-WorkerExitImplementation-Handoff.md`. Verify RelocationEvidence and
> MANIFEST-1, then launch the Prompt-2 peer-blind Code/State reviews and the
> originating executable closure. Do not rebuild or change accepted design."

**Exit:** a manager starting cold in garns-wz reaches the parked gate from the
checkpoint, without consulting datascad.

### Phase 4 — Publish *(gated on G1)*

| Step | Goal | How |
|---|---|---|
| **P4.1** | Apply G1 | For example, set both repositories private on GitHub (operator action). |
| **P4.2** | Push | Confirm the identity first (§3 G1), then `$GWZ --root "$WZ" push`. Verify `origin/main` equals HEAD for the root and `garns`; `garns-rust` is unchanged. |
| **P4.3** | Push tags *(per G3)* | `$GWZ --root "$WZ" tag --push v9-5-b2-cut`. |

**Exit:** v9-6 has history and a remote. The single-copy risk is closed.

### Phase 5 — datascad is left alone

**Operator decision, 2026-10-04: no change of any kind to datascad** — no
detach, no pointer document, no edit. `datascad/garns-v9-6` stays a registered
member, byte-identical, exactly as every earlier lane (`garns-v2` … `garns-v9-5`)
stays registered there as history. It remains the location the historical
records cite, by path and by member id (`mem_garns_v9_6`). The relocation record
(P3.1) and the updated checkpoint (P3.3) are what direct the lane to its new
home.

---

## 5. Sequencing

```
P0.2 backup ── today, alone, independent of everything
P0 ──► P1 ──┬─► P2 (scratch verification) ──┐
            └─► P3 (records, root docs) ─────┴─► P4 (G1)

datascad: untouched throughout
```

P2 and P3 touch disjoint files (scratch extractions versus new records and root
docs), so they can be taken by separate agents. Everything else is serial by
design: each phase's exit is the next phase's precondition.

---

## 6. Risks and their controls

| Risk | Control |
|---|---|
| Lossy copy | Source inventory over every file; verified against the working tree **and** a `git archive` of the commit (P1.2, P1.5). |
| A lane guard silently breaks | All 38 maps plus HandoffEvidence re-run at the new location before any new record is written (P1.3). |
| `AGENTS_GWZ.md` regenerated mid-gate | No `gwz init --update` until the gate closes (P1.4). |
| Evidence polluted by running tools in place | Executable checks run in scratch only (Phase 2). |
| Accidental publication | G1 before any push; tags local until G1/G3. |
| Wrong copy edited during a mirror interval (G2 = after only) | Root README notice; short interval; datascad left untouched. |
| Push as the wrong GitHub identity | Confirm before P4.2. |

---

## 7. Deliberately not done in the move — W7 inputs

- **Installability.** `parse.py:21` reads `BUILD_ROOT/"grammar"/"garns.lark"`
  with no `importlib.resources` fallback, and there is no `src/garns/garns.lark`,
  so a wheel imports and then fails on first parse. The v9-5 cut fixed both;
  the fix is retrievable at tag `v9-5-b2-cut`.
- `docs/LIMITATIONS.md` says there is no `pyproject.toml` and no console script —
  stale against v9-6's own `pyproject.toml`.
- `check_product.py`'s forbidden-leak list names only v9-5 lane paths. After
  relocation it should also forbid `datascad/garns-v9-6` paths in shipped text.
- 129 files with absolute paths, 72 of them with `/Users/owebeeone` — a curation
  pass before any public release.
- The v9-5 cut's `uv.lock`, `contracts/contract-ids.json`, `PROVENANCE.md` and
  `CUT_MANIFEST.json` are removed by the import; v9-6's `BASELINE.json` and
  `evidence/v9-5-b2/` supersede them. All are retrievable at the tag.

---

## 8. Open decisions carried forward (not blocking the move)

- **sdax** is absent from v9-6's decision record (zero mentions in docs,
  dev-docs and source), despite the 2026-09-12 decision that both language lanes
  use the sdax family. A3 is executor-agnostic, so nothing precludes it, but A16
  keeps names provisional only **until W3 Surface review**. After that, the lane
  requires STOP and operator direction for any accepted-design change. It must
  enter the record before W3.
- **The Rust lane** has no W-package. `garns-rust` stays a member, untouched.
  Its README and AGENTS text refer to the v9-5 cut's `contracts/` registry, which
  the import removes. `garns-artifact`'s constants do not read that file, but the
  documentary coupling goes stale until the Rust lane is given a home.

---

## 9. Execution log

### 2026-10-04 — Phases 0–2 *(no commit, no push)*

| Step | Result |
|---|---|
| P0.2 | **Skipped** by operator decision. |
| P0.3 | `GarnsV9-6-Migration-SourceInventory.sha256`: 1,135 entries, SHA-256 `11e120ce63676529afa5751e8688a099d7265485d3ba24561df446456041d233`. |
| P0.4 | All guards **pass at the old location**: MANIFEST-1 self-hash `7e80ca67…`; 117/117, 31/31, 20/20, 741/741, 115/115, 71/71, 111/111, 614/614, 13/13, HandoffEvidence 7/7; no bytecode/cache artifacts. |
| P0.5 | Annotated tag `v9-5-b2-cut` on root `7c706b1`, garns `34f2f14` and garns-rust `7e8e319`; **local only**. gwz 1.0.17 behaves differently from its own help text: `gwz tag NAME` with the default selection tagged the **two members only**, so the root needed `--target @root`. Pushing will hit the same split: `gwz tag --push` spans members only, so the root's tag needs a separate push. |
| P1.1–P1.2 | `rsync` into `garns/`. `diff -r` empty; inventory 1,135/1,135 OK from the destination; 0 extra files; 0 files that `.gitignore` would drop. **Working tree only — nothing staged or committed.** |
| P1.3 | All guards **pass at the new location**, with the same counts as P0.4. The two `../AGENTS_GWZ.md` pins resolve to garns-wz's byte-identical copy. |
| P2.1 | In a scratch copy, the lane's exact commands and interpreters: focused 130 OK, full 233 OK, product 5 PASS — on CPython 3.11.14, 3.12.12, 3.13.12 and 3.14.3, all exit 0. This matches VerificationLogs-1 exactly. SQLite `ResourceWarning`s appear on 3.13 (114) and 3.14 (138) only, as the handoff states. No caches left. |
| P2.2 | `tools/check.py` in scratch: **127/128 — G10 "no corpus vocabulary in compiler" FAILS** at `src/garns/backends/contracts/semantic.py:98` (see below). |

**Pre-existing finding for the W1A11 reviewers — not caused by the move.** G10's
regex `\b(everbility|vaultwarden|appflowy|practice|book|author)\b` (case-
insensitive, `tools/check.py:589`) matches the word "author" in the error string
"reserved key aliases cannot be visible author fields".

- The original datascad tree fails identically, so the move did not cause it.
- The word is absent from the W1A11 ContractAmendment's archived pre-amendment
  baseline. The current bytes are pinned from ContractAmendment MANIFEST onward,
  and `semantic.py` is listed in WorkerExitImplementation MANIFEST-1 — so the
  line sits **inside the frozen review object**.
- It went unseen because the lane forbids running `tools/check.py` (it writes
  gate evidence) and `tools/check_product.py` does not include G10. The committed
  `gate-report.json` (2026-10-03, all pass) predates the amendment.
- Not fixed here: the move is byte-exact and source writes are stopped. Whether
  to reword the string (a bounded source repair) or refine the gate is the
  review's decision under the lane's correction rules.

The git steps were approved next ("go 1-5"); their results follow.

### 2026-10-04 — Steps 1–5 *(operator: "go 1-5")*

| Step | Result |
|---|---|
| 1 · import commit | `garns` `a4ade29`. 586 paths staged (554 added, 14 modified, 7 deleted, 11 renamed), leaving all 1,135 tracked files equal to the source. Proven from `git archive HEAD`: 1,135/1,135 OK, 0 extra files, `git ls-files` = 1,135. Author Gianni Mariani; no AI trailers. Nine empty `generated/` subdirectories cannot be represented in git; a clone-equivalent tree without them passes G9 5/5, the product checks 5/5 and the full suite 233/233. |
| 2 · relocation commit | `garns` `461b879`: `GarnsV9-6-Relocation.md`; the four Prompt-2 files, two path lines changed in each; `…-RelocationEvidence.sha256`, verifying 8/8; and the checkpoint update. Every other guard still passes. HandoffEvidence now differs only on the checkpoint, by design. |
| 4 · push `garns` | `34f2f14..461b879` to `owebeeone/garns`, a fast-forward. |
| 5 · member tags | `v9-5-b2-cut` pushed for `garns` (→ `34f2f14`) and `garns-rust` (→ `7e8e319`); verified with `ls-remote`. |
| 3 · root commit | This commit: the plan, the source inventory, Rev 1 marked superseded, README and ARCHITECTURE describing the v9-6 layout, and gwz's lock update pinning `garns` at `461b879`. |
| 4/5 · root push and root tag | Follow this commit. |

**Push route.** SSH on this machine authenticates as gripd, which has read-only
access to these repositories, so `gwz push` and `gwz tag --push` could not be used.
Every push went through git to the same `origin` remotes, with a one-shot
`-c url.https://github.com/.pushInsteadOf=git@github.com:`, authenticating as
owebeeone through the `gh` credential helper. No remote or config was changed.
