# Tasks: A Persona's Life in the Studio

**Input**: Design documents from `/specs/004-sonavida-persona-life/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: required. Principle VII makes tests-first mandatory for gate logic, the Studio Link and access control; here that is the gate path (no piece crosses without an accepted verdict), the Studio Link use, erasure, and the read-only team view. The proposal guard of R-1 is tested before proposals exist, because it is the one place code shapes a persona's choice.

**Organization**: grouped by user story, so each can be built and checked on its own.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependency between them)
- **[Story]**: US1 to US6 from spec.md

## Path Conventions

- **Component**: `components/sonavida/`, the `JomarJunior/sonavida` repository. Paths below are relative to it.
- **Hub**: contracts in `specs/004-sonavida-persona-life/contracts/`; the ModelMora contract in `specs/002-modelmora-inference/contracts/modelmora-v1.yaml`.
- **Synthetic only**: every fixture, test persona and scripted run uses synthetic personas (FR-037). Resident definitions are never read by these tasks.

---

## Phase 1: Setup

- [x] T001 Add `pyproject.toml` in `components/sonavida/` for Python 3.12 managed with `uv`: package `sonavida` under `src/`, console entry point `sonavida`, dependencies `miraveja-persona` and `miraveja-studiolink` pinned to git commits, `httpx`, `pydantic` v2, `anyio`; dev group `pytest`, `ruff`, `mypy`
- [x] T002 [P] Add `README.md` headed **🎭 SonaVida**: Studio-only, no network listener, memory never leaves the Studio, links to `specs/004-sonavida-persona-life/` in the hub
- [x] T003 [P] Configure ruff, ruff-format and mypy strict over `src` and `tests`, and the `pytest` layout `tests/unit`, `tests/integration`, `tests/privacy`
- [x] T004 Extend `.github/workflows/guard.yml` into `ci.yml` in `components/sonavida/`: the existing `miraveja-persona guard`, then lint, format, mypy and the test suite, checking the hub out at a pinned `main` commit into `MIRAVEJA_HUB_PATH`
- [x] T005 [P] Add `tests/conftest.py`: a scratch `SONAVIDA_HOME` per test, a scratch vault built from the spec 003 hub synthetic examples (Pellam Quist, Ivo Marrowfield), and an autouse fixture that blocks every socket except the in-process stand-ins; fixtures are synthetic only (FR-037) except the in-process stand-ins

---

## Phase 2: Foundational (blocking)

- [x] T006 Implement `src/sonavida/ports/clock.py`: `Clock` protocol, `RealClock` (Studio local time) and `SimulatedClock` (seeded, advances instantly to the next due turn) per R-6 and FR-039
- [x] T007 [P] Implement `src/sonavida/ports/models.py`: `Models` protocol and the `ModelMoraClient` over loopback HTTP against `modelmora-v1.yaml` (submit, ask, collect result, availability), returning typed outcomes `result`, `busy(retry_at)`, `starting`, `stopping`, `failed`, `cannot-serve`; the client refuses any host other than loopback (Principle V), tested
- [x] T008 [P] Implement `src/sonavida/standins/models.py`: a scripted `Models` stand-in returning deterministic text and tiny PNGs, and scriptable *starting*, *busy*, *stopping*, *failed* answers (FR-038)
- [x] T009 [P] Define the remaining ports in `src/sonavida/ports/`: `perception.py` (one neutral description per image), `gate.py` (Verdict with accepted, reason, labels, feedback), `studiolink.py` (announce presence, hand over candidate, collect and acknowledge experiences and erasure notices, wrapping `miraveja-studiolink`), `vault.py` (`load_resident`, `load_synthetic`, public names via `pasts.list_pasts`)
- [x] T010 Write `tests/unit/test_memory_store.py`: entries append-only with monotonic `id`; `importance` "integer 1–5"; `kind` restricted to the data-model list; no API deletes a row (FR-025, FR-026); FTS recall returns relevant entries ordered by match, recency and importance within a budget
- [x] T011 Implement `src/sonavida/memory/store.py`: the SQLite schema of data-model.md (`self`, `entries` with `visitor_pseudonym` and `visitor_name` columns, `pieces`, `attempts`, `inbox`, FTS5 `entries_fts(text, reason)`), append and recall (R-2), one file per persona at `$SONAVIDA_HOME/personas/<persona-id>/memory.sqlite`
- [x] T012 [P] Implement `src/sonavida/translate.py`: `Models` outcomes to persona situations per R-7 ("the studio is not ready; it may be ready around <time of day>", "the work was interrupted", "the attempt did not come out"); never a model name, code or queue position (FR-031)
- [x] T013 [P] Write `tests/unit/test_translate.py`: every outcome maps to its situation, and no output contains a model name, a digit-only queue position, or any of the ModelMora refusal codes

**Checkpoint**: memory, time, ports and translation exist and are tested.

---

## Phase 3: User Story 1 — A persona comes alive and keeps its own hours (Priority: P1) 🎯 MVP

**Goal**: a persona is born once from its definition, keeps its own hours, and announces every presence change.

**Independent test**: a synthetic persona runs several simulated days against the reference stand-in: born once, every chosen presence change announced and explained (spec US1).

- [x] T014 [P] [US1] Write `tests/unit/test_birth.py`: birth copies self-knowledge into `self`, seeds `seed` entries and a `self-aware` entry, renders shared pasts in the first person with `{n}` as "I" or the other participant's public name ("someone I once knew" if unknown) per R-4; a second start never reads the definition, even if the file changed (FR-002); synthetic-as-resident, outside-vault, not-born, changed-since-birth and author's-note definitions are refused with their codes (FR-003)
- [x] T015 [US1] Implement `src/sonavida/birth.py` per R-3 and R-4, from the definition alone with no per-persona code (FR-001)
- [x] T016 [P] [US1] Write `tests/unit/test_proposals.py`: for every reachable state, the proposal set equals the set of actions valid in that state per `contracts/turn-protocol.md`; `set-presence`, `do-nothing` and `leave-the-museum` are always present; hints only annotate and never remove (R-1, FR-005, FR-015)
- [x] T017 [US1] Implement `src/sonavida/turns/proposals.py` with hints drawn only from the persona's `self` tendencies and memory
- [x] T018 [P] [US1] Write `tests/unit/test_reply.py`: replies are validated against the turn-protocol schema (action valid in state, exactly its details, `reason` required, `importance` 1–5, `nextTurnIn` any ISO 8601 duration); one retry with the validation message; a second failure becomes `lost-thread` and a one-hour rest (R-10); the prompt's "where I am" section carries the current date and time and how long since the persona was last in the studio (FR-006)
- [x] T019 [US1] Implement `src/sonavida/turns/reply.py` and `src/sonavida/turns/prompt.py`: the prompt builder of R-10 (self, situation, recalled entries one by one in time order, proposals, schema)
- [x] T020 [US1] Implement `src/sonavida/actions/presence.py` and `src/sonavida/actions/nothing.py`: `set-presence` announces through `StudioLink` and remembers with reason (FR-007); `do-nothing` is remembered as `chose-nothing`
- [x] T021 [US1] Implement `src/sonavida/life.py`: the turn loop, next turn at the persona's chosen time, `time-away` on return after the Studio was off (FR-009)
- [x] T022 [P] [US1] Write `tests/integration/test_hours.py`: a simulated three-day run of the Pellam example against the reference stand-in: one birth, every chosen presence change announced and in memory with its reason (SC-003), , time away remembered after a simulated Studio outage, and a second start of the same persona while it is alive refused as `already_alive` (FR-004); a SIGTERM while personas are alive announces every one of them as away before the process exits (FR-008, SC-003)
- [x] T023 [US1] Implement `src/sonavida/runtime.py`: host personas as tasks, per-persona `lock` via `fcntl.flock` refusing `already_alive` (FR-004), orderly SIGINT/SIGTERM shutdown announcing each persona away before stopping (FR-008)
- [x] T024 [US1] Implement `src/sonavida/cli.py` `run` (real and `--simulate DAYS --seed N --standins`) and `status` per `contracts/cli.md`

**Checkpoint**: MVP. A persona lives and keeps hours.

---

## Phase 4: User Story 2 — The persona makes a piece and says what it is (Priority: P1)

**Goal**: intention, attempts seen through perception, finish with title and statement, or abandon.

**Independent test**: a synthetic persona's waking period with the models stand-in yields pieces each with intention, attempts, title and statement, all in memory (spec US2).

- [ ] T025 [P] [US2] Write `tests/unit/test_creation_actions.py`: `form-intention`, `make-attempt`, `look-again`, `rework`, `finish` (requires a kept attempt, sets `title` and `statement`), `abandon`; each records its entry and reason; piece state moves only along the data-model transitions; every image request carries the persona's `ask` verbatim and adds only words from its own `craft` and `stylesAndMedia`, never subject, style or intent of the runtime's choosing (FR-011)
- [ ] T026 [US2] Implement `src/sonavida/actions/creation.py` and the one general mapping from the persona's words and craft preferences to `Models` image requests (FR-010, FR-012, FR-014, FR-016), storing attempt images under the persona directory
- [ ] T027 [P] [US2] Implement `src/sonavida/standins/perception.py`: the default stand-in asking `Models` for a neutral description of the image, and a scripted one for tests (R-9); the persona sees its work only through this port (FR-013)
- [ ] T028 [US2] Write `tests/integration/test_making.py`: a simulated waking period produces finished pieces with intention, attempts and what was seen, a title and a statement, and an abandoned piece with its reason; choosing not to work is allowed and remembered (FR-015)

---

## Phase 5: User Story 3 — The persona decides whether to show its work (Priority: P1)

**Goal**: keep or submit; only the gate can send a piece on, with its labels.

**Independent test**: with the gate stand-in scripted, only submitted pieces reach it and only accepted ones reach the Studio Link stand-in, with the gate's labels (spec US3).

- [ ] T029 [P] [US3] Write `tests/integration/test_showing_work.py` first: kept, abandoned and rejected pieces never reach the Studio Link stand-in; accepted ones arrive as candidates with title and statement unchanged and the gate's labels; the persona's suggested labels reach the gate; the gate may add labels and the persona cannot remove them; rejections return as `verdict` memories with feedback (SC-004, FR-018, FR-020, FR-042)
- [ ] T030 [P] [US3] Write `tests/privacy/test_single_path.py`: a static check that the only code that calls `StudioLink.hand_over_candidate` is the `AiGate` stand-in, and that nothing from `self`, intentions, memory or reasoning is included in a candidate (FR-018, FR-021)
- [ ] T030a [P] [US3] Write `tests/integration/test_no_bypass.py` first: `sonavida run` without `--simulate --standins` and without a configured real AI gate refuses to start with `gate_not_configured`, and the gate stand-in rejects any Studio Link target that is not the in-process reference stand-in (Principle III, R-9)
- [ ] T031 [US3] Implement `src/sonavida/standins/gate.py`: accepted/rejected with reason, labels = persona suggestion plus scripted additions, feedback addressed to the persona on rejection, and hand-over of accepted candidates through `StudioLink` (R-9, FR-019); pre-alpha default accepts and keeps the persona's labels, clearly logged as a stand-in, and bound to the in-process reference stand-in only (R-9)
- [ ] T032 [US3] Implement `src/sonavida/actions/showing.py`: `submit` with optional `suggestedLabels` (`explicit`, `violent`) and `keep`, each remembered with its reason (FR-017)

---

## Phase 6: User Story 4 — The persona remembers what happened to its work (Priority: P1)

**Goal**: experiences one by one, no numbers, erasure removes identity only.

**Independent test**: scripted experiences and an erasure notice on the reference stand-in become memories once each, with no counts, and the erased visitor is forgotten (spec US4).

- [ ] T033 [P] [US4] Write `tests/integration/test_experiences.py`: each scripted experience becomes one `experience` entry in delivery order, acknowledged only after it is stored, none lost or repeated across a restart (SC-005, FR-022, FR-023); a visitor met before is recognized by pseudonym and name (FR-024); a refused message leaves no trace (FR-028); a human-gate outcome moves the piece to `exhibited`, `declined` or `taken-down` and becomes a memory with the reason addressed to the persona where one exists (US4 scenario 5)
- [ ] T034 [P] [US4] Write `tests/privacy/test_no_metrics.py`: with twenty reactions on one piece, no prompt or memory entry produced by **🎭 SonaVida** contains a count, total, average, rank, rating, trend, comparison or money (FR-027, SC-005)
- [ ] T035 [P] [US4] Write `tests/privacy/test_erasure.py`: after an erasure notice, both visitor columns are cleared, every `⟨v:pseudonym⟩` token and any free-typed display name is replaced by "someone", the memories remain, the raw file bytes after `VACUUM` contain neither pseudonym nor name, the notice was acknowledged only after this, and the notice is not a memory; pieces are untouched (FR-032, FR-033, SC-006)
- [ ] T036 [US4] Implement `src/sonavida/inbox.py`: collect and acknowledge experiences and erasure notices through `StudioLink`, store through the memory store, tokens for visitors (R-8), and piece state updates from human-gate outcomes
- [ ] T037 [US4] Implement `src/sonavida/memory/erasure.py` per R-8
- [ ] T038 [US4] Render recalled experiences in `src/sonavida/turns/prompt.py` strictly one by one, tokens to names, never aggregated (R-10)

---

## Phase 7: User Story 5 — The team can read a persona's life (Priority: P2)

**Goal**: read-only, plain-language, on the Studio only.

**Independent test**: a team member reads a simulated week and answers the fixed questions (spec US5).

- [ ] T039 [P] [US5] Write `tests/privacy/test_read_only.py`: `sonavida memory` and `sonavida pieces` open the file with `mode=ro`; no CLI command or public function writes to, edits or deletes from memory; there is no network listener anywhere in the package (FR-035, FR-036)
- [ ] T040 [US5] Implement `src/sonavida/memory/reader.py` and the `memory` and `pieces` commands in `src/sonavida/cli.py` per `contracts/cli.md` and R-12
- [ ] T041 [US5] Add `tests/fixtures/reading-questions.md`: ten fixed questions of the kind in SC-002, with how to check each answer against a run

---

## Phase 8: User Story 6 — The Studio's limits are part of the persona's life (Priority: P2)

**Goal**: waits and interruptions as persona behavior, never lost or reduced.

**Independent test**: with the models stand-in scripted *busy*, *starting*, *stopping*, no intention is lost or reduced and memory has no technical terms (spec US6).

- [ ] T042 [P] [US6] Write `tests/integration/test_studio_limits.py`: *busy* and *starting* give `studio-not-ready` with a time of day and the persona chooses what to do; *stopping* leaves the piece `interrupted`, and `continue-unfinished` is proposed on return (FR-030); *failed* gives `attempt-failed`; the request the persona asked for is never lowered, retried or replaced by **🎭 SonaVida** (FR-029, SC-007)
- [ ] T043 [US6] Implement the waiting and interruption handling in `src/sonavida/life.py` and `src/sonavida/actions/creation.py`, using `translate.py`

---

## Phase 9: Several personas and leaving (cross-story, P1 requirements FR-040, FR-041)

- [ ] T044 [P] Write `tests/integration/test_several.py`: three synthetic personas alive at once for a simulated week, each with its own memory file and hours; zero entries of one in another's memory or pieces; every intention carried out, set aside or waiting (FR-041, SC-010); the run completes in under 5 minutes (plan performance goal)
- [ ] T045 [P] Write `tests/integration/test_leaving.py`: one `leave-the-museum` choice gives `thinking-of-leaving`; a second consecutive one records `departed` with its reason, announces away once, releases the lock, marks memory read-only, and `sonavida run` refuses it as `departed` afterwards; a non-consecutive second choice does not count (FR-040, R-11)
- [ ] T046 Implement `src/sonavida/actions/leaving.py` and the departed handling in `src/sonavida/runtime.py`

---

## Phase 10: Polish & Cross-Cutting

- [ ] T047 [P] Write `tests/privacy/test_logs.py`: with a marker phrase in every prose field of a synthetic persona, a full simulated week writes the marker to no log line, error or file outside `$SONAVIDA_HOME` (FR-034, SC-008)
- [ ] T048 [P] Write `tests/integration/test_week.py`: a seeded simulated week of the Pellam example finishes at least one piece and completes in under 5 minutes (SC-001 mechanics, plan performance goal); a second synthetic definition comes alive with no code change (SC-009)
- [ ] T049 [P] Update `docs/components/sonavida.md` in the hub: the turn model, memory location, ports, and that resident personas must be frozen before they can come alive
- [ ] T050 [P] Record the spec 004 decisions in the hub's `docs/foundation/FOUNDATION-LOG.md`: departure recorded in the Studio; erasure removes identity, not memories; several personas each alone; labels suggested by the persona and decided by the gate; hybrid turns; SQLite with full-text recall
- [ ] T051 Run every scenario in `quickstart.md` except the human reading trial and record the outcome in `specs/004-sonavida-persona-life/checklists/quickstart-run.md`
- [ ] T052 Run the SC-002 reading trial with a team member who did not watch the run (needs a person) and record it in `specs/004-sonavida-persona-life/checklists/sc-002-trial.md`
- [ ] T053 Run `/speckit-converge` and repeat implement and converge until it reports converged

---

## Dependencies

```text
Setup ──► Foundational ──► US1 (MVP) ──► US2 ──► US3
                              │            └──► US6
                              ├──► US4
                              └──► US5
US1 + US2 + US3 ──► Phase 9 (several, leaving) ──► Polish
```

- US1 builds the turn loop, proposals, prompts and runtime that every other story uses.
- US3 needs US2's finished pieces; US6 needs US2's attempts.
- US4 and US5 need only US1's memory and life loop.

## Parallel examples

- **Foundational**: T007, T008, T009, T012 and T013 together.
- **US1**: T014, T016 and T018 (tests) together before T015, T017, T019.
- **US3**: T029 and T030 together before T031 and T032.
- **US4**: T033, T034 and T035 together before T036 to T038.
- **Polish**: T047 to T050 together.

## Implementation strategy

1. **MVP**: Phases 1 to 3. A synthetic persona is born and keeps its own hours.
2. **Making and showing**: US2 and US3, so pieces exist and cross only through the gate.
3. **Memory of the world**: US4, with no metrics and real erasure.
4. **The team's view and the Studio's limits**: US5 and US6.
5. **Several and leaving**, then polish, the quickstart run and converge.

Commits in `components/sonavida/` reference spec 004.
