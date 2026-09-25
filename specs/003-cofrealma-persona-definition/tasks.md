# Tasks: Resident Persona Definition Format

**Input**: Design documents from `/specs/003-cofrealma-persona-definition/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: required. Principle VII makes tests-first mandatory for access control, which here is the loaders' refusals (FR-022, FR-032) and the public-repository guard (FR-021). The rule catalog is measured on a labeled corpus (SC-003, SC-007, SC-008), so each rule family's corpus comes before its rules.

**Organization**: grouped by user story, so each can be built and checked on its own.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependency between them)
- **[Story]**: US1 to US5 from spec.md

## Path Conventions

- **Hub** (this repo): `specs/003-cofrealma-persona-definition/contracts/` holds the schemas, rule catalog, interface and synthetic examples, and is the source of truth.
- **Library**: `components/miraveja-persona/`, the `JomarJunior/miraveja-persona` repository, created and checked out in the hub's `components/`. Paths marked *library* are relative to it.
- **Synthetic only**: every fixture, corpus file and example is a synthetic persona marked `nature: synthetic`. No resident persona, name or seed memory appears anywhere in these tasks' output (Principle VIII, SC-006).

---

## Phase 1: Setup

**Purpose**: a publishable, empty library that can read the hub's schemas.

- [X] T001 Add `pyproject.toml` in `components/miraveja-persona/` for Python 3.12, package `miraveja_persona` under `src/`, console entry point `miraveja-persona`, runtime dependencies `ruamel.yaml`, `jsonschema`, `pydantic` v2 only (no network library), and a `dev` group with `pytest`, `ruff`, `mypy`
- [X] T002 [P] Add `README.md` in `components/miraveja-persona/` headed `miraveja-persona` (a library, no brand emoji), stating that the format is public and every resident definition stays private, listing the commands from `contracts/cli.md`, and linking to `specs/003-cofrealma-persona-definition/` in the hub
- [X] T003 [P] Configure ruff, ruff-format and mypy (strict, over `src` and `tests`) in `components/miraveja-persona/pyproject.toml`, and the `pytest` layout `tests/corpus`, `tests/loader`, `tests/guard`, `tests/vault`, `tests/privacy`, `tests/cli`
- [X] T004 Copy the hub's `persona-definition-v1.schema.json` and `author-note-v1.schema.json` into `components/miraveja-persona/src/miraveja_persona/schema/` as package data (R-3)
- [X] T005 Write `components/miraveja-persona/tests/test_schema_drift.py`: the bundled schemas are byte-identical to the hub's files at `$MIRAVEJA_HUB_PATH/specs/003-cofrealma-persona-definition/contracts/`, and the hub's `contracts/examples/*.yaml` validate against them
- [X] T006 Add `.github/workflows/ci.yml` in `components/miraveja-persona/`: lint, format, mypy and the full test suite, checking the hub out at a pinned commit (not `main`) into `MIRAVEJA_HUB_PATH`

**Checkpoint**: an empty but publishable library whose schemas cannot drift from the hub's.

---

## Phase 2: Foundational (blocking)

**Purpose**: safe reading, the models, findings and the privacy fixture every story needs.

- [X] T007 Write `components/miraveja-persona/tests/privacy/conftest.py` and `test_no_network.py`: an autouse fixture that makes any socket connection raise, applied to the whole suite, and a test that importing and running every CLI command under it succeeds (FR-016)
- [X] T008 Implement `components/miraveja-persona/src/miraveja_persona/yamlio.py`: read exactly one YAML 1.2 document in safe mode, keeping line numbers per node; refuse duplicate keys, anchors, aliases, custom tags and multiple documents; keep every scalar a string except integers and booleans the schema needs (so `writtenOn` and `when` stay strings)
- [X] T009 [P] Write `components/miraveja-persona/tests/test_yamlio.py`: each refusal in T008 yields a `structure.yaml` finding with a line number; a date-shaped value stays a string
- [X] T010 Implement `components/miraveja-persona/src/miraveja_persona/model.py`: frozen pydantic v2 models `Definition`, `Identity`, `Taste`, `Voice`, `Tendencies`, `LoreName`, `SeedMemory`, `SharedPast` and `AuthorNote` mirroring the schemas, with `extra="forbid"` and the bounds verbatim from data-model.md: `publicName` "string, 1–80"; `when` "prose, 1–200"; prose max 4000 chars; `truth` max 8000; slug `^[a-z0-9-]{1,64}$`; UUID pattern; `participants` "list of UUID, 2+", unique; `themes`, `cares`, `seedMemories` 1+; `openlyAI` constant `true`; `telling` in `agreed | intended-difference`; `nature` in `resident | synthetic`
- [X] T011 [P] Implement `components/miraveja-persona/src/miraveja_persona/findings.py`: the `Finding` record (`rule`, `part` as JSON Pointer, `line`, `quote`, `cites`, `certainty` certain|uncertain, `decided`), a stable part key that addresses list items by their `id` or `story`, never by position (e.g. `/sharedPasts[junior-regatta]/happened`), the fingerprint `<rule>:<key>:<first 8 hex of SHA-256 of quote>`, and text and JSON renderers matching `contracts/cli.md`
- [X] T012 Implement `components/miraveja-persona/src/miraveja_persona/validate.py`: schema validation with the bundled Draft 2020-12 schemas, one `structure.schema` finding per error with its JSON Pointer and line, plus `structure.version` (naming supported versions), `structure.ids`, `structure.self-in-participants` and `structure.placeholder` from `contracts/check-rules.md`
- [X] T013 [P] Write `components/miraveja-persona/tests/test_validate.py`: the hub's three examples produce no finding; a missing required part, an unknown field (including an `authorNote` field inside a definition), `openlyAI: false`, a 81-character `publicName`, a duplicate seed memory id, a shared past without the persona's own id, and `{3}` with two participants each produce exactly the expected rule; `pellam-quist.persona.yaml` holds every required and optional part of the schema, so the hub has one example that uses the whole format (FR-023)

**Checkpoint**: any file can be read safely and checked for structure; findings print as the interface says.

---

## Phase 3: User Story 1 — Write a new persona in under an hour (Priority: P1) 🎯 MVP

**Goal**: a team member scaffolds, writes and checks a new definition, entirely on their own machine.

**Independent test**: with only the format and the synthetic example, a team member writes a new synthetic persona and `check` reports it valid in under an hour (SC-001; quickstart 1 and 2).

- [X] T014 [P] [US1] Write `components/miraveja-persona/tests/cli/test_new.py`: `new --name "Test Persona Nine" --synthetic` writes a file with a fresh UUID, every required and optional part, a guidance comment for each, `nature: synthetic`; it refuses to overwrite; guidance comments are absent from the loaded model; two runs give two different UUIDs
- [X] T015 [US1] Implement `components/miraveja-persona/src/miraveja_persona/scaffold.py` and the `new` command, with guidance lines modeled on the hub's `pellam-quist.persona.yaml` (R-11)
- [X] T016 [P] [US1] Write `components/miraveja-persona/tests/cli/test_check_basic.py`: `check` on the examples exits 0; on a scaffold exits 1 naming each empty required part; unreadable file exits 2; `--format json` emits a list of Findings; output for a clean file is empty
- [X] T017 [US1] Implement `components/miraveja-persona/src/miraveja_persona/check.py` and `cli.py` for `check PATH... [--format text|json]` with exit codes 0 valid, 1 certain finding, 3 undecided uncertain findings only, 2 usage (from `contracts/cli.md`), wiring structure validation and a rule registry that later phases fill
- [ ] T018 [US1] Time SC-001 once with a team member who has not seen the format, writing a new synthetic persona from `new` and the hub example; record the time and any part that was unclear in `specs/003-cofrealma-persona-definition/checklists/sc-001-trial.md`, and fix unclear guidance in `scaffold.py`

**Checkpoint**: MVP. A definition can be written and structurally checked.

---

## Phase 4: User Story 2 — A runtime brings the persona to life from the definition alone (Priority: P1)

**Goal**: **🎭 SonaVida** loads a definition with no other input; resident personas load only from a marked vault and only once born.

**Independent test**: `load_synthetic` returns every part of each example with no other input, swapping examples changes only data, and `load_resident` accepts only a born resident definition inside a marked vault (SC-002; quickstart 6 and 8).

- [X] T019 [P] [US2] Write `components/miraveja-persona/tests/loader/test_load_synthetic.py`: returns a frozen `Definition` exposing identity, taste, voice, tendencies, cares, craft, self-image, lore, seed memories and shared pasts with `{n}` placeholders intact; refuses `nature: resident` (`resident_outside_studio`), an author's note (`author_note`), an unsupported version (`unsupported_version`, naming supported versions) and an invalid file (`invalid`)
- [X] T020 [P] [US2] Write `components/miraveja-persona/tests/loader/test_load_resident.py` in a scratch vault: refuses a synthetic definition (`synthetic_as_resident`), a path outside a `.cofrealma`-marked directory (`outside_vault`), a definition with no ledger entry (`not_born`), one whose hash differs from its ledger entry (`changed_since_birth`), an author's note (`author_note`); accepts a born, unchanged resident definition
- [X] T021 [P] [US2] Write `components/miraveja-persona/tests/privacy/test_refusal_messages.py`: every `DefinitionRefused` carries a reason code and a message with no prose from the file, proven with a unique marker phrase in every prose field (Principle VIII)
- [X] T022 [US2] Implement `components/miraveja-persona/src/miraveja_persona/load.py`: `load_synthetic(path)` and `load_resident(path, cofrealma_root)` with the refusal codes from `contracts/cli.md`, no logging of content, and a `Definition` with no reload, merge or update method (FR-018, FR-022, FR-032, R-10)
- [X] T023 [P] [US2] Write `components/miraveja-persona/tests/vault/test_freeze.py`: `freeze` adds an entry with `id`, `publicName`, `definition` path relative to the vault root, `sha256`, `bornOn`; refuses a synthetic definition, one already frozen, and one failing `check --tree`; the ledger is append-only
- [X] T024 [US2] Implement `components/miraveja-persona/src/miraveja_persona/vault.py`: locate the vault root by its `.cofrealma` marker, read and append `ledger/births.yaml`, hash definition files, and the `freeze PATH --tree ROOT` command (R-8)
- [X] T025 [P] [US2] Write `components/miraveja-persona/tests/vault/test_vault_rules.py`: `vault.frozen-changed`, `vault.frozen-missing`, `vault.reused` (identifier and public name, departed ledger entries included), `vault.note-inside` and `vault.synthetic-in-vault` each fire as specified in `contracts/check-rules.md`
- [X] T026 [US2] Add the vault-wide rules to `vault.py` and `check --tree ROOT` to `cli.py`
- [X] T027 [US2] Write `components/miraveja-persona/tests/loader/test_self_sufficient.py`: a stand-in runtime starts both hub examples through `load_synthetic` alone and exposes every data-model field; swapping one example for the other changes no code path (SC-002)

**Checkpoint**: **🎭 SonaVida** (spec 004) can build against the loader.

---

## Phase 5: User Story 3 — Definitions cannot order a persona around or carry numbers (Priority: P1)

**Goal**: the rule catalog in `contracts/check-rules.md` flags orders, metrics, hard lines, human claims, forbidden content and feelings in shared pasts, and lists every shared past.

**Independent test**: on the labeled corpus, 100% of seeded violations are flagged and clean definitions get at most one false finding each (SC-003, SC-007, SC-008; quickstart 3, 4 and 5).

### Corpus first

- [X] T028 [US3] Create the labeled corpus format in `components/miraveja-persona/tests/corpus/README.md` and `conftest.py`: each case is a synthetic definition plus an `expect.yaml` listing the expected rule ids and parts; the runner reports seeded-violation recall and false findings per clean case
- [X] T029 [P] [US3] Add corpus cases in `tests/corpus/orders/` for `order.cadence`, `order.lifespan`, `order.subject`, `order.modal` and `order.future-conduct`, including tendencies that must pass ("tends to work late", "rarely exhibits") and past acts that must pass ("has never admitted")
- [X] T030 [P] [US3] Add corpus cases in `tests/corpus/metrics/` for `metric.money`, `metric.count`, `metric.number`, in every prose field and in seed memories
- [X] T031 [P] [US3] Add corpus cases in `tests/corpus/hardlines/` for `hardline.living-artist-style`, `hardline.real-name`, `hardline.likeness` and `hardline.minors`, using only invented names; declared lore names and the persona's own public name must pass
- [X] T032 [P] [US3] Add corpus cases in `tests/corpus/openly-ai/` for `ai.human-claim`, with human-shaped pasts that must pass ("grew up above her father's print shop")
- [X] T033 [P] [US3] Add corpus cases in `tests/corpus/pasts/` for `past.feeling`, `past.motive`, `past.feeling-ambiguous`, and multi-definition cases for `past.unknown-participant`, `past.possible-mistake` and `past.telling-mismatch`, including the quickstart 4 table
- [X] T034 [P] [US3] Add corpus cases in `tests/corpus/forbidden/` for `forbidden.visitor`, `forbidden.model` and `forbidden.secret`, and a `tests/corpus/clean/` set of at least ten varied clean synthetic definitions
- [X] T035 [US3] Add `components/miraveja-persona/tests/corpus/test_corpus.py`: fails unless recall is 100% on seeded violations and no clean case has more than one false finding (SC-003, SC-007, SC-008)

### Rules

- [X] T036 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/orders.py` with pattern files for the `order.*` rules, scoped to the fields the catalog names
- [X] T037 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/metrics.py` with pattern files for the `metric.*` rules
- [X] T038 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/hardlines.py`: proper-name detection per R-5 (capitalized runs not at a sentence start, not in a common-word list including "I", "AI", months and weekdays, not a declared lore name, not the persona's own public name), plus the style, likeness and minors rules
- [X] T039 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/openly_ai.py` for `ai.human-claim`
- [X] T040 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/pasts.py` for `past.feeling`, `past.motive` and `past.feeling-ambiguous` on shared pasts and on seed memories naming another persona's identifier
- [X] T041 [P] [US3] Implement `components/miraveja-persona/src/miraveja_persona/rules/forbidden.py` for `forbidden.visitor`, `forbidden.model` and `forbidden.secret`
- [X] T042 [US3] Register every rule in `check.py`; apply to author's notes only `structure.*`, `hardline.*`, `metric.money`, `forbidden.visitor` and `forbidden.secret` (FR-033)
- [X] T043 [US3] Implement the cross-definition shared-past checks in `vault.py` per the data-model.md table: one-sided passes; all `agreed` and identical passes; all `agreed` and differing is uncertain `past.possible-mistake`; all `intended-difference` passes; mixed markings is certain `past.telling-mismatch`; differing participants is certain unless all are `intended-difference`; unknown participant is certain `past.unknown-participant` (R-7)

### Decisions and the shared-past listing

- [X] T044 [P] [US3] Write `components/miraveja-persona/tests/cli/test_decide.py`: `decide` records `finding`, `decision` (`accepted` | `not-an-issue`), `by`, `on` in the `decisions.yaml` beside the definition; refuses certain findings; a decided finding is reported as decided and `check` exits 0; a decision matching no current finding is reported stale; reordering shared pasts or seed memories keeps every decision valid (FR-017, R-6)
- [X] T045 [US3] Implement `components/miraveja-persona/src/miraveja_persona/decisions.py` and the `decide` command
- [X] T046 [P] [US3] Write `components/miraveja-persona/tests/cli/test_pasts.py`: over a scratch vault holding both hub examples, `pasts` lists `junior-regatta` as agreed with one version and `lighthouse-mural` as an intended difference with both versions, each participant shown by UUID and public name, in text and JSON, in under one second (FR-028, SC-007)
- [X] T047 [US3] Implement `components/miraveja-persona/src/miraveja_persona/pasts.py` and the `pasts ROOT` command

**Checkpoint**: the full rule catalog, measured; the spec 007 baseline can be listed.

---

## Phase 6: User Story 4 — Private by design (Priority: P1)

**Goal**: every public repository blocks definitions, author's notes and the vault marker unless synthetic, without ever printing their content.

**Independent test**: in scratch git repositories, the guard blocks a resident definition, an unmarked one, a resident author's note, a pasted signature line and a `.cofrealma` marker, and passes the hub's examples (SC-004; quickstart 7).

- [X] T048 [P] [US4] Write `components/miraveja-persona/tests/guard/test_guard.py` using scratch git repositories: blocks `nature: resident`; blocks a document with the signature but no `nature` or an unknown `nature`; blocks an author's note marked resident; blocks any `*.persona.yaml` or `*.note.yaml` not marked synthetic; blocks a `.cofrealma` file; blocks a line starting `miravejaPersona:` or `miravejaAuthorNote:` inside a Markdown file; passes the hub's three examples; scans git-tracked files by default; exits 0, 1 or 2 as in `contracts/cli.md`
- [X] T049 [P] [US4] Write `components/miraveja-persona/tests/privacy/test_guard_output.py`: with a unique marker phrase in every prose field of each blocked file, guard output (stdout and stderr) contains file, line and reason only, and never the marker (R-9)
- [X] T050 [US4] Implement `components/miraveja-persona/src/miraveja_persona/guard.py` and the `guard [PATH...]` command (R-9)
- [X] T051 [US4] Add `.github/workflows/guard.yml` to the hub: install `miraveja-persona` from its repository at a pinned commit and run `miraveja-persona guard` on every push and pull request
- [X] T052 [P] [US4] Add `miraveja-persona guard` to `components/miraveja-persona/.github/workflows/ci.yml`, so the library guards itself
- [X] T053 [P] [US4] Add a `miraveja-persona guard` step beside the existing guard in `components/modelmora/.github/workflows/ci.yml` (spec 002 T006), pinned to a library commit; commit in `modelmora` referencing spec 003
- [X] T054 [P] [US4] Add a `miraveja-persona guard` step beside the existing guard in `components/miraveja-studiolink/.github/workflows/ci.yml` (spec 001 T005), pinned to a library commit; commit in `miraveja-studiolink` referencing spec 003
- [X] T054a [P] [US4] Add `.github/workflows/guard.yml` running `miraveja-persona guard` (pinned to a library commit) to `components/museumusa/`; commit in `museumusa` referencing spec 003 (FR-021, SC-004)
- [X] T054b [P] [US4] Add the same guard workflow to `components/portaguarda/`; commit referencing spec 003
- [X] T054c [P] [US4] Add the same guard workflow to `components/curagusta/`; commit referencing spec 003
- [X] T054d [P] [US4] Add the same guard workflow to `components/sonavida/`; commit referencing spec 003
- [X] T054e [P] [US4] Add the same guard workflow to `components/descridiva/`; commit referencing spec 003
- [X] T055 [US4] Document the private vault in `docs/components/cofrealma.md`: layout (`.cofrealma` marker, `personas/<slug>/definition.persona.yaml`, `decisions.yaml`, `*.note.yaml`, `ledger/births.yaml`), a CI workflow that installs the library and runs `check --tree .`, and the rule that the vault is never checked out under a public tree. Writing the vault's own files is done in the private repository, not by these tasks

**Checkpoint**: a resident definition cannot enter any public repository unnoticed.

---

## Phase 7: User Story 5 — The persona outgrows its definition (Priority: P2)

**Goal**: nothing in the format or the library can pull a living persona back toward its definition.

**Independent test**: a reviewer confirms every part of the format is described as a starting point, and the library offers no way to re-apply a definition to a living persona (spec US5).

- [X] T056 [P] [US5] Write `components/miraveja-persona/tests/loader/test_no_reapply.py`: `Definition` is frozen and exposes no update, merge, diff-against-memory or reload method; the public API (`miraveja_persona.__all__`) contains nothing that compares a persona to its definition
- [X] T057 [US5] Review the `description` of every part in both hub schemas and every scaffold guidance line so each reads as who the persona is at the start, never a rule it must keep, and cites the requirement and the principle or Charter article it serves, so a reviewer can trace every part in under 15 minutes (SC-005); fix wording in `specs/003-cofrealma-persona-definition/contracts/*.schema.json` and `scaffold.py`, then re-copy the schemas (T004) so T005 passes

**Checkpoint**: all five stories done.

---

## Phase 8: Polish & Cross-Cutting

- [X] T058 [P] Add `docs/components/miraveja-persona.md` in the hub: boundaries (owns the format's tools; never owns persona content), consumers (**🎭 SonaVida**, **🔐 CofreAlma** CI, every public repository's guard), the rules that bite hardest (I, II, VIII), repository and specs
- [X] T059 [P] Add `miraveja-persona` to `docs/foundation/ECOSYSTEM-MAP.md` under shared assets and generic libraries, beside `miraveja-studiolink`
- [X] T060 [P] Add to `docs/foundation/GLOSSARY.md`: **Synthetic persona**, **Shared past**, **Author's note**, **Self-image**, **Birth ledger**, using the spec's definitions
- [X] T061 [P] Record the spec 003 decisions in `docs/foundation/FOUNDATION-LOG.md`: birth seed only; no model names; shared pasts as facts, never feelings; intended differences, lies as past acts and author's notes; a past held openly as an AI; optional self-image; definitions frozen for good; YAML with a closed schema; the `miraveja-persona` library
- [X] T062 Run every scenario in `specs/003-cofrealma-persona-definition/quickstart.md` against a scratch vault and record the outcome in `specs/003-cofrealma-persona-definition/checklists/quickstart-run.md`
- [ ] T063 Run `/speckit-converge` and repeat implement and converge until it reports converged

---

## Dependencies

```text
Phase 1 Setup ──► Phase 2 Foundational ──► US1 (MVP) ──► US2 ─┬─► US4
                                               │               │
                                               └──► US3 ◄──────┘ (T043 needs vault.py from US2)
                                                                 US5 after US2 (loader) and US1 (scaffold)
                                                                 Polish after all
```

- US1 builds `check` and the rule registry that US3 fills.
- US2's `vault.py` is needed by US3's cross-definition checks (T043) and `pasts` (T047), and by `freeze`'s pre-check.
- US4's guard is independent of the rules and can start right after Phase 2; its CI tasks (T051 to T054e) need the library pushed and each target repository attached to the session.
- US5 needs the loader (US2) and the scaffold (US1).

## Parallel examples

- **Phase 2**: T009, T011 and T013 together once T008 and T010 are in.
- **US2**: T019, T020, T021, T023 and T025 (all tests) together, before T022, T024 and T026.
- **US3**: the corpus tasks T029 to T034 together; then the rule modules T036 to T041 together.
- **US4**: T048 and T049 together before T050; then T052 to T054e together.
- **Polish**: T058 to T061 together.

## Implementation strategy

1. **MVP**: Phases 1 to 3. A team member can scaffold and structurally check a definition. Stop and run the SC-001 trial (T018).
2. **Runtime-ready**: add US2, so spec 004 can start building against `load_resident` and `load_synthetic`.
3. **Rules**: add US3, measured on the corpus before it is trusted.
4. **Leak-proof**: add US4 and roll the guard out to every public repository.
5. **Freedom check and polish**: US5, then Phase 8, then converge.

Commits in `components/miraveja-persona/` (and in `modelmora`, `miraveja-studiolink`, `museumusa`, `portaguarda`, `curagusta`, `sonavida` and `descridiva` for T053 to T054e) reference spec 003. Pushing to `miraveja-persona` needs the Claude GitHub App installed on that repository.

---

## Phase 9: Convergence

- [ ] T064 CRITICAL: extend `components/miraveja-persona/src/miraveja_persona/guard.py` so `guard` also blocks secrets in every scanned text file, reusing the `forbidden.secret` patterns from `rules/forbidden.py` and printing only file, line and "secret-shaped value" (never the value); add cases to `tests/guard/test_guard.py` and `tests/privacy/test_guard_output.py`; then bump the pinned `miraveja-persona` commit in the hub's `.github/workflows/guard.yml` and in the guard steps of `modelmora`, `miraveja-studiolink`, `museumusa`, `portaguarda`, `curagusta`, `sonavida` and `descridiva` per Constitution VIII (contradicts)
- [ ] T065 Add `components/miraveja-persona/tests/loader/test_carried_note.py`: `load_synthetic` and `load_resident` refuse a definition that carries an author's note field (`truth`, `authorNote`, `miravejaAuthorNote`) per FR-032, SC-008 (partial)
- [ ] T066 Reconcile the rule-pattern location: patterns live in `components/miraveja-persona/src/miraveja_persona/rules/*.py`, so update `specs/003-cofrealma-persona-definition/contracts/check-rules.md` ("the library's pattern files") and the `rules/` line in `plan.md` to say the rule modules hold the complete, tested lists per plan: project structure (partial)
- [ ] T067 Add `components/miraveja-persona/tests/test_performance.py`: `check` on one hub example under 2 seconds, `check --tree` over a scratch vault of 20 synthetic definitions under 10 seconds, and `guard` over the library repository under 10 seconds per plan: Performance Goals (partial)
