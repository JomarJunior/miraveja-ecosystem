# Tasks: Studio Link Contract

**Input**: Design documents from `/specs/001-studiolink-contract/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: required. FR-039 says contract tests are written before either end, and Principle VII makes test-first mandatory for the contract.

**Organization**: grouped by user story, so each can be built and checked on its own.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependency between them)
- **[Story]**: US1 to US5 from spec.md

## Path Conventions

- **Hub** (this repo): `specs/001-studiolink-contract/contracts/`
- **Library**: `components/miraveja-studiolink/`, a separate public repository checked out under `components/`
- Library paths below are relative to `components/miraveja-studiolink/`

---

## Phase 1: Setup

**Purpose**: create the library repository and make it publishable.

- [x] T001 Create the public repository `JomarJunior/miraveja-studiolink` (Apache-2.0) and clone it to `components/miraveja-studiolink/`. Requires the Visionary's go-ahead, since it is visible beyond this machine.
- [X] T002 Add `pyproject.toml` for Python 3.12 with the package `miraveja_studiolink`, and a console entry point `miraveja-studiolink`
- [X] T003 [P] Add `README.md` headed **🔗 miraveja-studiolink**, stating that the contract itself lives in the hub and linking to `specs/001-studiolink-contract/`
- [X] T004 [P] Configure linting, formatting and type checking, and a `pytest` layout with `tests/schemas`, `tests/client`, `tests/standin`, `tests/conformance`
- [X] T005 [P] Add the CI workflow: lint, type check, tests, plus the guard that fails if a persona definition, a real persona name or a secret appears in the repository (Principle VIII, SC-008)
- [X] T006 Read the contract from the hub checkout at `../../specs/001-studiolink-contract/contracts/studiolink-v1.yaml`, overridable with `STUDIOLINK_CONTRACT_PATH`; CI checks the hub out at a pinned commit rather than copying the file, so the hub stays the single source of truth (FR-008, R-13)

**Checkpoint**: an empty but publishable library that can read the contract.

---

## Phase 2: Foundational (blocking)

**Purpose**: everything both ends need. No user story can start before this is done.

- [X] T007 Write `tests/schemas/test_examples.py`: every file in the hub's `contracts/examples/valid/` validates and every file in `invalid/` is rejected, with `_`-prefixed keys stripped. This is what enforces the rules with no code of their own: no tone or conduct judgment on a comment (FR-028), no rejected verdict crossing (FR-029), and no meeting kind from the Museum side (FR-042) (FR-035)
- [X] T008 Implement `src/miraveja_studiolink/messages/` as Pydantic v2 models for PersonaRef, VisitorRef, Verdict, PresenceAnnouncement, Candidate, PersonaComment, Acknowledgement, Experience (three kinds), ExperienceBatch, ErasureNoticeBatch, ExhibitionView and Refusal, all forbidding extra fields (data-model.md)
- [X] T009 Write `tests/schemas/test_drift.py`: the models' generated schemas match the hub's `studiolink-v1.yaml`, and the test fails when either side changes alone (R-13)
- [X] T010 [P] Implement `src/miraveja_studiolink/messages/refusal.py`: the closed reason list, with no reason expressing tone or conduct, and a refusal exception carrying reason, detail and supported versions (FR-028, FR-032, R-10)
- [X] T011 [P] Implement strict response validation in `src/miraveja_studiolink/client/strict.py`: any unknown field in anything from the Museum side rejects the whole message (FR-023)
- [X] T012 Implement `src/miraveja_studiolink/client/transport.py`: the Studio's bearer credential, TLS, the `/studiolink/v1` base path, and one place where every request is made (FR-004, R-3, R-9)

**Checkpoint**: messages exist, are provably in step with the hub, and nothing loose can get in.

---

## Phase 3: User Story 1 — Build and test the Studio end alone (P1) 🎯 MVP

**Goal**: a persona's whole day works against the stand-in, with no Museum side anywhere.

**Independent test**: `pytest tests/client` passes with no **🏛️ MuseuMusa** or **🛡️ PortaGuarda** installed (SC-001).

- [X] T013 [US1] Write `tests/client/test_presence.py` and `tests/standin/test_presence.py` first: announcing presence is recorded and reported (FR-009, FR-010)
- [X] T014 [US1] Implement the stand-in skeleton in `src/miraveja_studiolink/standin/app.py`: an in-memory ASGI app serving every path in the contract, with a scripting API to seed state, an inspection API exposing everything the Studio has sent (candidates, comments, presence, acknowledgements), and a reset to a clean state (FR-036)
- [X] T015 [US1] Implement `client/presence.py` and the stand-in's presence handler
- [X] T016 [US1] Write tests, then implement `client/candidates.py` and the stand-in's candidate handler: multipart hand-over, accepted verdict required, unknown label refused, lands in the human-gate queue and never in the exhibition (FR-012 to FR-014)
- [X] T017 [US1] Write tests, then implement `client/experiences.py` and the stand-in's experience queue: collect with cursor and limit, comment/reaction/gate-outcome kinds, oldest first, acknowledge (FR-016 to FR-019)
- [X] T018 [US1] Write tests, then implement `client/comments.py` and the stand-in's comment handler: hard-lines verdict required, published into the same conversation, refusals for `piece_not_on_display` and `conversation_closed` (FR-025 to FR-027, FR-030, FR-032)
- [X] T019 [US1] Write tests, then implement `client/exhibition.py` and the stand-in's exhibition view: named looking persona, time order, no counts, changes nothing, becomes no one's experience (FR-040, FR-041, FR-043)
- [X] T019a [US1] Write tests, then implement the piece-image exchange in `client/images.py` and the stand-in: an image is fetched by piece identifier through the contract, and the view hands out no location outside it (FR-040a, R-14)
- [X] T020 [US1] Implement remembered refusals in `client/memory.py`: a refusal the Studio receives is handed to the caller as something the persona may remember, and never reveals a visitor's choice (FR-033, R-11)
- [X] T021 [US1] Implement the scripting fixtures in `tests/fixtures/`, including `one-visitor-comment.yaml` used by quickstart Scenario 2, with synthetic personas only, plus a test asserting that every fixture and example uses a synthetic persona and no real name (Principle VIII, SC-008)
- [X] T022 [US1] Add the `miraveja-studiolink standin` command so the stand-in can be run from a terminal (quickstart Scenario 2)

**Checkpoint**: Stage A can begin. Roadmap 002 to 004 have something to build against.

---

## Phase 4: User Story 2 — Nothing that counts or pays reaches a persona (P1)

**Goal**: the promise the contract exists to keep.

**Independent test**: `pytest tests/client -k metrics` (SC-003).

- [X] T023 [US2] Write `tests/client/test_metrics_refused.py`: the stand-in is told to answer with each contaminated example, and every one is refused with nothing passed to the persona (FR-021 to FR-023)
- [X] T024 [US2] [P] Write `tests/client/test_individual_events.py`: twelve reactions arrive as twelve experiences with their own visitor and time, and no summary anywhere (FR-016, spec US2 scenario 3)
- [X] T025 [US2] Add a static review test that walks every schema reachable from the Museum side in the hub's contract and fails on any field whose name or description suggests a count, score, rank, view figure or amount. This checks the contract as written (FR-038)
- [X] T026 [US2] [P] Write `tests/standin/test_no_metrics_emitted.py`: the stand-in never emits such a field at runtime, whatever it is asked for and however much activity it is seeded with. This checks behavior rather than the document, so a future end cannot pass T025 and still leak (FR-038)

**Checkpoint**: Principle II is enforced by tests on both ends, not by good intentions.

---

## Phase 5: User Story 3 — Build and test the Museum end alone (P2)

**Goal**: any Museum end can be checked with no Studio running.

**Independent test**: the conformance suite against the stand-in passes 100% (SC-002).

- [X] T027 [US3] Write the conformance suite's own tests first: it must fail a deliberately broken Museum end (a double-delivery, a missing refusal, a count in a response)
- [X] T028 [US3] Implement `src/miraveja_studiolink/conformance/suite.py`: plays the Studio's side of every exchange with synthetic personas and reports pass or fail per exchange (FR-037)
- [X] T029 [US3] Add the `miraveja-studiolink conformance --target --credential` command (quickstart Scenario 5)
- [X] T030 [US3] Run the suite against the stand-in in CI and require 100% (SC-002, spec US3 scenario 4)

**Checkpoint**: roadmap 009 can build the Museum end against a real check.

---

## Phase 6: User Story 4 — The Studio keeps hours and loses nothing (P2)

**Goal**: an offline Studio costs nothing but time.

**Independent test**: `pytest tests/client -k offline` (SC-004).

- [X] T031 [US4] Write tests, then implement at-least-once delivery: an interrupted collection delivers the same items again, and acknowledged items never return (FR-019)
- [X] T032 [US4] Write tests, then implement send-mark handling on the stand-in for candidates and comments: a repeat returns the original answer and creates nothing; the same text under a new mark creates a second one (FR-015, FR-031, FR-031a, SC-005)
- [X] T033 [US4] [P] Write tests, then implement withdrawal: an experience whose comment or reaction is removed before collection is never delivered (FR-020)
- [X] T034 [US4] [P] Write tests, then implement the staleness window in the stand-in: a persona unheard from beyond the window reads as away, default 2 hours, configurable (FR-011, R-12)
- [X] T035 [US4] Write tests, then implement the separate erasure-notice queue with its own cursor and acknowledgement, collectable on its own, never presented as an experience (FR-044 to FR-047, SC-009)
- [X] T036 [US4] [P] Write tests, then implement receipt-time ordering when the Studio's clock is wrong (Edge Cases, FR-018)
- [X] T036a [US4] [P] Write `tests/client/test_no_reachability.py`: no schema, parameter or header in the contract names a callback, address, port or availability window for the Studio, and no exchange is defined that the Museum side could start (FR-001, FR-003)
- [X] T036b [US4] Write tests, then implement holding in the stand-in: everything meant for a persona stays until that persona's Studio collects it, with no Studio ever present during the test, and nothing expires in version 1 (FR-002)
- [X] T037 [US4] Write tests, then implement per-persona pseudonyms in the stand-in: stable for one persona, different across personas, opaque and irreversible to the Studio (FR-017, FR-017a, SC-010, R-6)

**Checkpoint**: the walking skeleton survives a real machine that sleeps.

---

## Phase 7: User Story 5 — The contract can change without breaking either end (P3)

**Goal**: version 2 will be possible without a broken afternoon.

**Independent test**: `pytest tests/client -k version` (SC-006).

- [X] T038 [US5] Write tests, then implement `GET /studiolink/versions` in the stand-in and `client/versions.py`, carrying no persona data (FR-007)
- [X] T039 [US5] Write tests, then implement refusal of an unsupported version before any content is acted on, naming the supported versions (FR-005, FR-006, SC-006)

---

## Phase 8: Polish

- [X] T040 [P] Walk every scenario in `quickstart.md` on a clean checkout and correct anything that does not behave as written (SC-007)
- [X] T041 [P] Write the library's usage docs: how a Studio component talks to the Museum side, and how a Museum end is checked
- [X] T042 Release version 1.0.0 of the library: tag `v1.0.0` and publish a GitHub release from the tag. Consumers install from the tag; publishing to a package index is a later decision, since nothing outside the ecosystem needs it yet (Principle IX)
- [X] T043 Record in `docs/foundation/FOUNDATION-LOG.md` that contract version 1 is implemented, and close spec 001

---

## Dependencies

- **Phase 1 → Phase 2 → everything else.**
- **US1 (Phase 3)** is the MVP. **US2 (Phase 4)** needs the stand-in and client from US1.
- **US3 (Phase 5)** needs the stand-in (T014) but nothing from US2.
- **US4 (Phase 6)** deepens the behavior US1 introduced; T031 to T037 touch the same queue code, so they are sequential unless marked [P].
- **US5 (Phase 7)** is independent of US2 to US4 and can be done any time after Phase 2.
- **Phase 8** last.

## Parallel opportunities

- T003, T004, T005 after T002.
- T010 and T011 alongside T008.
- Within US2: T024 and T026 alongside T023.
- Within US4: T033, T034, T036 and T036a alongside T031 and T032.

## Amendments

2026-09-22, after `/speckit-analyze`: T006, T007, T010, T014, T021, T025, T026, T042 reworded; T019a (piece image), T036a (no reachability) and T036b (holding) added. 46 tasks in total.

## Traceability

| Requirement group | Tasks |
|---|---|
| Direction and availability (FR-001 to FR-004) | T012, T014, T031, T036a, T036b |
| Versioning (FR-005 to FR-008) | T006, T009, T038, T039 |
| Presence (FR-009 to FR-011) | T013, T015, T034 |
| Candidates (FR-012 to FR-015) | T016, T032 |
| Experiences (FR-016 to FR-020) | T017, T031, T033, T036 |
| Looking at the museum (FR-040 to FR-043) | T019, T019a |
| Erasure (FR-044 to FR-047) | T035 |
| Memory, never metrics (FR-021 to FR-024) | T023, T024, T025, T026 |
| Persona voice (FR-025 to FR-033) | T018, T020, T032 |
| Privacy (FR-017, FR-017a, FR-034) | T037, T008 |
| Testability (FR-035 to FR-039) | T007, T009, T021, T027 to T030 |
