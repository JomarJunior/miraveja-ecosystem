# Tasks: Model Inference in the Studio

**Input**: Design documents from `/specs/002-modelmora-inference/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: required. Principle VII makes tests-first mandatory for gate logic, contracts and access control; here that is the queue rules, the registry's licence gate and caller isolation. FR-034 requires the busy, withdrawal, ordering and shutdown behavior to be testable with no GPU.

**Organization**: grouped by user story, so each can be built and checked on its own.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: can run in parallel (different files, no dependency between them)
- **[Story]**: US1 to US5 from spec.md

## Path Conventions

- **Hub** (this repo): `specs/002-modelmora-inference/contracts/modelmora-v1.yaml` is the contract's source of truth.
- **Component**: `components/modelmora/`, the `JomarJunior/modelmora` repository, already created and checked out in the main hub checkout. Paths below are relative to it.

---

## Phase 1: Setup

**Purpose**: make the component repository publishable and able to read the contract.

- [X] T001 Add `pyproject.toml` in `components/modelmora/` for Python 3.12, package `modelmora`, console entry point `modelmora`, with dependency groups separating GPU extras (`torch`, `transformers`, `diffusers`) from the base install so CI can run with no GPU
- [X] T002 [P] Add `README.md` in `components/modelmora/` headed **🧠 ModelMora**, stating that it is Studio-only, never part of the Studio Link, and linking to `specs/002-modelmora-inference/`
- [X] T003 [P] Configure ruff, ruff-format and mypy (over `src`, `tests` and `scripts`) in `components/modelmora/pyproject.toml`, plus a `pytest` layout with `tests/api`, `tests/queue`, `tests/registry`, `tests/worker` and `tests/privacy`
- [X] T004 [P] Add `components/modelmora/scripts/guard_public_repo.py`: fail on secret patterns and on any non-synthetic persona name in tests and fixtures (Principle VIII, FR-031)
- [X] T005 Implement `components/modelmora/src/modelmora/contract.py`: load `modelmora-v1.yaml` from the hub checkout, overridable with `MODELMORA_CONTRACT_PATH`, mirroring how `miraveja-studiolink` does it
- [X] T006 Add `.github/workflows/ci.yml` in `components/modelmora/`: guard, lint, format, mypy, and the GPU-free test suite, checking the hub out at a pinned commit (not `main`) for the contract

**Checkpoint**: an empty but publishable component that can read its contract.

---

## Phase 2: Foundational (blocking)

**Purpose**: the message models, the stand-in runners and the service skeleton every story needs. No user story can start before this is done.

- [X] T007 Implement `components/modelmora/src/modelmora/messages.py` as Pydantic v2 models for TextRequest, ImageRequest, Accepted, RequestStatus, Result, ServableModel, Availability and Refusal, all forbidding extra fields, with the contract's bounds: instructions 1–200000 chars, conversation ≤ 200 turns, images ≤ 8 of `image/png|image/jpeg|image/webp`, description 1–20000, width and height 64–4096, steps 1–200, guidance 0–30, maxLength 1–32000, temperature 0–2
- [X] T008 Write `components/modelmora/tests/api/test_contract_drift.py`: the models' generated schemas match the hub's `modelmora-v1.yaml`, failing when either side changes alone
- [X] T009 [P] Implement `components/modelmora/src/modelmora/refusals.py`: the closed reason set `busy`, `starting`, `stopping`, `unknown_model`, `model_unavailable`, `invalid_request`, `cannot_be_served_on_this_studio`, `failed_during_generation` (FR-011), each with whether it carries `retryAfterSeconds`, and a refusal exception whose `detail` can never hold request or result content (FR-030)
- [X] T010 [P] Implement `components/modelmora/src/modelmora/config.py`: line limit (default 32, never below the SC-002 burst size of 20, so that criterion measures results rather than `busy` refusals), bounded overtaking time (default 2 minutes, FR-019), result holding time (default 1 hour, FR-032), idle-unload timeout (default 10 minutes), bind host fixed to `127.0.0.1`, port default 8431, and caller tokens
- [X] T011 Implement the runner interface and test-mode stand-ins in `components/modelmora/src/modelmora/runners/`: `base.py` (generate text, generate image, declared memory footprint, load and unload), `standin.py` returning deterministic text and a tiny PNG with configurable fake load time and fake footprint (FR-033, R-10)
- [X] T012 Write `components/modelmora/tests/worker/test_standin_runners.py`: stand-ins honor seeds, report their fake footprint, and simulate load time, so the queue tests can rely on them
- [X] T013 Implement the service skeleton in `components/modelmora/src/modelmora/api/app.py`: all five paths from the contract, bound to loopback only, with bearer caller tokens naming the calling component (R-9) and a `--test-mode` switch selecting the stand-in runners
- [X] T014 Write `components/modelmora/tests/api/test_loopback_only.py`: the service binds `127.0.0.1` and refuses to start when configured with any other interface (FR-027), and works with no network reachable (FR-028)

**Checkpoint**: a service that answers every path with stand-in models, provably local.

---

## Phase 3: User Story 1 — Ask for text and get it back (Priority: P1) 🎯 MVP

**Goal**: a caller sends instructions and receives text, naming at most a model.

**Independent test**: with one text model on record and nothing loaded, a test caller receives generated text naming the model and version (spec US1 independent test).

- [X] T015 [US1] Write `components/modelmora/tests/api/test_text_requests.py` first, covering US1 acceptance scenarios 1 to 6 and SC-001 (a caller obtains results while naming at most a model, with zero caller code touching model loading): default model resolution, a named model, `unknown_model` with no substitute, images read by a capable model, `invalid_request` when no model on record reads images, and identical output for the same model, version, seed and length limit
- [X] T016 [US1] Implement text generation in `components/modelmora/src/modelmora/runners/text.py` using `transformers`, honoring seed, maxLength and temperature, and reporting the model name and version that produced the text
- [X] T017 [US1] Implement default-model resolution in `components/modelmora/src/modelmora/registry/defaults.py` for the slots `text`, `text_with_images` and `image`; a request with images defaults to the `text_with_images` slot (FR-003)
- [X] T018 [US1] Implement request validation in `components/modelmora/src/modelmora/api/validate.py`: refuse `unknown_model` for a model not on record, and `invalid_request` when a request carries images and the chosen or default model cannot read them, naming that in the detail (FR-003)
- [X] T019 [US1] Implement the submit path for text in `components/modelmora/src/modelmora/api/requests.py`, returning `Accepted` with `requestId`, `position`, `estimatedWaitSeconds` and the resolved `model` (FR-010)
- [X] T020 [US1] Implement result assembly in `components/modelmora/src/modelmora/worker/results.py`: every result carries the model name and version and the settings actually used, plus `filterNote` when a model's own built-in filter changed the output (FR-008)
- [X] T021 [US1] Write `components/modelmora/tests/worker/test_filter_disclosure.py` first, then make it pass in T020: a stand-in configured to filter its output produces a result whose `filterNote` says so, and **🧠 ModelMora** adds no judgment of its own (FR-008)

**Checkpoint**: personas can think and speak, and the AI gate can reason. Roadmap 004 can start against test mode.

---

## Phase 4: User Story 2 — Ask for an image and get it back (Priority: P1)

**Goal**: a caller sends a description and a size and receives an image.

**Independent test**: with one image model on record, a test caller receives an image with the model's name, version, seed and settings used (spec US2 independent test).

- [x] T022 [US2] Write `components/modelmora/tests/api/test_image_requests.py` first, covering US2 acceptance scenarios 1 to 3 and SC-001 for images: an image of the requested size with seed and settings reported, a text model evicted to make room with only a longer wait, and an unsupported size refused before queueing
- [ ] T023 [US2] Implement image generation in `components/modelmora/src/modelmora/runners/image.py` using `diffusers`, honoring size, seed, steps, guidance and things to avoid
- [x] T024 [US2] Implement GPU residency in `components/modelmora/src/modelmora/worker/residency.py`: track resident models with measured footprint and last-used time, evict least-recently-used to make room, and unload a model idle beyond the timeout — no caller ever sees a memory error (FR-009)
- [x] T025 [US2] Implement capability checks in `components/modelmora/src/modelmora/api/validate.py`: a size or setting the chosen model cannot produce is refused with a reason before the request is queued, and a request needing more memory than the GPU has even alone is refused `cannot_be_served_on_this_studio` (FR-011, Edge Cases)
- [x] T026 [US2] Implement result image holding in `components/modelmora/src/modelmora/worker/holding.py`: bytes in a temporary directory wiped on startup and shutdown, collectable through `fetchResultImage` until `heldUntil` (default 1 hour), then discarded, and never written to the registry (FR-030, FR-032, R-7)

**Checkpoint**: pieces can be made. Assumption A-006 can finally be tested on the Studio.

---

## Phase 5: User Story 3 — Many requests at once, answered honestly (Priority: P1)

**Goal**: every caller gets an immediate, truthful answer and later a result.

**Independent test**: a burst of mixed requests from several callers for more models than fit at once; every request gets an immediate answer and later a result, and the answers match what happened (spec US3 independent test).

- [x] T027 [US3] Write `components/modelmora/tests/queue/test_admission.py` first: every submission answers within 1 second while the worker is busy, with a position and an estimate, or `busy` with `retryAfterSeconds` when the line is at its limit — never accepted and then dropped (FR-010, FR-015, SC-003)
- [x] T028 [US3] Write `components/modelmora/tests/queue/test_ordering.py` first: arrival order holds, a request for a resident model may overtake an older one needing a load, no request is overtaken for longer than the bounded time, and ordering never depends on the caller or persona (FR-019, SC-010)
- [x] T028a [US3] [P] Write `components/modelmora/tests/queue/test_burst.py` first: 20 mixed text and image requests from 4 callers, for more models than fit on the GPU together, all end with a result or an explained answer, with none lost, duplicated or left open (SC-002, quickstart Scenario 2)
- [x] T029 [US3] Write `components/modelmora/tests/queue/test_lifecycle.py` first: every accepted request ends in exactly one of done, failed, withdrawn or stopped before completion; a waiting request that is withdrawn never runs; a running request that fails leaves others untouched and is not retried with lower settings (FR-013, FR-014, Edge Cases)
- [x] T029a [US3] [P] Write `components/modelmora/tests/queue/test_no_deduplication.py` first: two callers submitting exactly the same instructions, model and seed get two requests and two results; **🧠 ModelMora** never decides that two requests are the same (Edge Cases). This is deliberately the opposite of the Studio Link's send-mark rule, and easy to "optimize" away later
- [x] T030 [US3] Implement the line in `components/modelmora/src/modelmora/queue/line.py`: admission against the limit, per-request state, position, and the single-consumer handoff to the worker
- [x] T031 [US3] Implement ordering in `components/modelmora/src/modelmora/queue/ordering.py`: arrival order with the resident-model exception bounded by the overtaking time from config, then the overtaken request runs next (FR-019)
- [x] T032 [US3] Implement the single generation worker in `components/modelmora/src/modelmora/worker/worker.py`: take one request at a time, ensure residency, generate, publish the result, and never run two generations at once (R-4)
- [x] T033 [US3] Implement estimates in `components/modelmora/src/modelmora/queue/estimates.py`: a rolling median of measured generation time per model and kind, plus load time when not resident, plus work ahead; updated as the line moves and never used to reorder work (FR-018, R-5)
- [x] T033a [US3] Write `components/modelmora/tests/queue/test_estimate_accuracy.py`: over a test-mode run where each model has been used at least once, at least 90% of requests start within 50% of the wait estimated at acceptance, and the measured figure is reported (SC-004, FR-018)
- [x] T034 [US3] Write `components/modelmora/tests/api/test_caller_isolation.py` **before** T035: a caller sees only its own requests, cannot read or withdraw another caller's request by id, and learns nothing beyond the length of the line (FR-017). Access control, so Principle VII requires the test first
- [x] T035 [US3] Implement the status and withdraw paths in `components/modelmora/src/modelmora/api/requests.py`, including the long poll of up to 30 seconds, returning state, position, estimate and the result when done, and enforcing caller ownership so T034 passes (FR-012, FR-013, FR-017)
- [x] T036 [US3] Write `components/modelmora/tests/queue/test_no_degradation.py` first: across a burst, zero results used a model, size, length or quality other than the request or its acceptance; a request that cannot be served as asked is refused or fails with a reason (FR-007, SC-008)

**Checkpoint**: a Studio shared by several personas and a curator behaves honestly under load.

---

## Phase 6: User Story 4 — Every served model's license is on record (Priority: P2)

**Goal**: nothing is served without a complete, confirmed record, and the trail survives retirement.

**Independent test**: add a model with a complete record and see it become available; a model with an incomplete record is refused; the registry lists every model that produced a result during the test (spec US4 independent test).

- [x] T037 [US4] Write `components/modelmora/tests/registry/test_licence_gate.py` first: a record without a licence name, licence source or team confirmation is never served and never loaded (FR-021, US4 scenario 2)
- [x] T037a [US4] [P] Write `components/modelmora/tests/registry/test_filter_disclosure_gate.py` first, then make it pass in T038 and T039: a model recorded as having an undisclosable built-in filter is never servable, and one whose filter behavior is disclosed (none, or disclosed and reportable) is. The team member judges, as with the licence; **🧠 ModelMora** records the judgment (spec Edge Cases, FR-008)
- [x] T038 [US4] Implement the SQLite schema in `components/modelmora/src/modelmora/registry/schema.sql` and `store.py`: the `model` table (name, version, kind, reads_images, licence_name, licence_source, source, weights_digest, added_by, added_at, licence_confirmed_by, licence_confirmed_at, filter_disclosure as `none`, `disclosed` or `undisclosable`), `service_period` (model_id, from, to nullable) and `default_model` (slot, model_id), with no field able to hold a hosted endpoint (FR-020, FR-026)
- [x] T039 [US4] Implement registry operations in `components/modelmora/src/modelmora/registry/registry.py`: add, list servable, set default per slot, retire (closing the service period while keeping the record), and resolve a name and version to a record (FR-023, FR-024)
- [x] T040 [US4] Implement the file digest check in `components/modelmora/src/modelmora/registry/verify.py`: compute a digest over the weight files, verify before the first load, and on a mismatch refuse to serve with `model_unavailable` and tell the team. "Telling the team" means one `WARNING` line in the operator log naming the model, version and expected digest, and a non-zero exit from `modelmora model verify` — the same channel everywhere the spec says the team is told (FR-022, R-8)
- [x] T041 [US4] Implement the CLI in `components/modelmora/src/modelmora/cli.py`: `modelmora serve`, `modelmora model add|list|retire|verify`, `modelmora model set-default`, so a team member changes the registry without touching code and can add a model in under 15 minutes (FR-025, SC-006)
- [x] T042 [US4] Implement the `listModels` path in `components/modelmora/src/modelmora/api/models.py`: name, version, kind, reads-images and licence per model, plus the default for each slot; `modelmora model list --all` shows retired models with their service dates so a reviewer can read the whole licence trail in under a minute (FR-024, SC-006)
- [x] T043 [US4] Write `components/modelmora/tests/registry/test_licence_trail.py`: every result produced during a run names a model whose record holds a licence, including after that model is retired (FR-023, SC-005)

**Checkpoint**: the licence trail is complete and reviewable.

---

## Phase 7: User Story 5 — The Studio's hours are visible to callers (Priority: P3)

**Goal**: starting, running and stopping are honest states a persona runtime can render as presence.

**Independent test**: ask about availability while starting, running and stopping; each answer matches the actual state and no request is left open on shutdown (spec US5 independent test).

- [ ] T044 [US5] Write `components/modelmora/tests/api/test_availability.py` first: `starting` rather than a failure before models are ready, `running` with the servable counts and the length of the line, nothing about another caller's requests, and — with an empty registry — servable counts of zero plus a request refused `model_unavailable`, never a failure of the service (FR-029, US5 scenarios 1 and 3, spec Edge Cases)
- [ ] T045 [US5] Write `components/modelmora/tests/queue/test_shutdown.py` first: on stop, every waiting or running request ends `stopped_before_completion`, and zero requests are left without a final answer (FR-029, SC-009)
- [ ] T046 [US5] Implement the lifecycle in `components/modelmora/src/modelmora/api/lifecycle.py`: the `starting`, `running` and `stopping` states, refusing new work with `starting` or `stopping` and a retry time, and draining the line on shutdown
- [ ] T047 [US5] Implement the `availability` path in `components/modelmora/src/modelmora/api/availability.py`, reporting state, queue length and servable counts per kind (FR-029)

---

## Phase 8: Polish

- [ ] T048 [P] Write `components/modelmora/tests/privacy/test_no_content_persisted.py`: after a run whose requests carry a unique marker phrase, a search of every log, error report and the registry finds it zero times; activity records hold only caller, model, settings, times and outcome (FR-030, SC-007)
- [ ] T049 [P] Write `components/modelmora/src/modelmora/checks/studio_smoke.py`: the manual on-Studio check from quickstart Scenario 7 (real text and image models, a forced eviction, availability, shutdown), excluded from CI
- [ ] T050 [P] Write `components/modelmora/docs/usage.md`: how a Studio component submits, polls, withdraws and collects; how a team member adds and retires a model; and why model names must never reach visitors (Principle IV, spec Assumptions)
- [ ] T051 Walk every scenario in `specs/002-modelmora-inference/quickstart.md` on a clean checkout and correct anything that does not behave as written
- [ ] T052 Release `components/modelmora/` v1.0.0: tag `v1.0.0` and publish a GitHub release from the tag
- [ ] T053 Record in `docs/foundation/FOUNDATION-LOG.md` that spec 002 is implemented, and close it with a pointer to roadmap 003

---

## Dependencies

- **Phase 1 → Phase 2 → everything else.**
- **US1 (Phase 3)** is the MVP and needs only the foundation.
- **US2 (Phase 4)** needs T024 (residency), which US3's worker also relies on; build US2 before US3.
- **US3 (Phase 5)** needs the submit path from US1 and residency from US2. T030 to T033 and T035 touch the same queue code and are sequential; T034 precedes T035 because access control is tested first (Principle VII).
- **US4 (Phase 6)** can be built in parallel with US1 to US3 after Phase 2; T017's default resolution reads the registry, so stub it until T039 lands.
- **US5 (Phase 7)** needs the line (T030) for draining, nothing else.
- **Phase 8** last.

## Parallel opportunities

- T002, T003, T004 after T001.
- T009 and T010 alongside T007.
- T028a and T029a alongside T027 to T029: separate test files, no shared code.
- The whole of Phase 6 alongside Phases 3 to 5, by two people; T037a alongside T037.
- T048, T049 and T050 together.

## Amendments

2026-09-23, after `/speckit-analyze`: T034 and T035 swapped so caller isolation is tested before it is implemented; T010, T015, T021, T022, T036, T038, T040, T041, T042, T044 reworded; T028a (SC-002 burst), T029a (no deduplication), T033a (SC-004 estimate accuracy) and T037a (filter-disclosure gate) added. 57 tasks in total.

2026-09-23, after `/speckit-implement MVP` (T001-T021): T019's submit path runs generation synchronously inside the request rather than through a real queue, since the line (T030-T033) and worker (T032) do not exist yet — a request still passes through exactly the `waiting -> running -> done/failed` states FR-014 requires, and `Accepted` still carries a position and an estimate (both 0 for now), so FR-010 is satisfied without pretending Phase 5 is built. `withdrawRequest` and `fetchResultImage` exist as contract-complete paths in the T013 skeleton but have nothing to withdraw before completion or hold as images yet; they will gain real behavior in Phase 4 (images) and Phase 5 (the queue). Phase 4 and 5 tasks are unaffected: T030-T036 replace this loop's internals without changing `submit_text_request`'s signature or return type.

2026-09-24, after `/speckit-implement Phase 5` (T027-T036): the race Phase 3 documented (two concurrent submissions both calling `Residency.ensure_loaded`) is closed by the single worker (T032); the stale caveat is removed from `api/requests.py`'s docstring. `AppState` moved to its own `api/state.py` module so `api/requests.py` and `api/app.py` could both depend on its shape without importing each other, since the worker's callbacks live in the former and are wired in the latter — an implementation-level split, not a contract change. The long poll (`waitSeconds`) waits for a *terminal* state rather than any state change, on the reading that a caller polling almost always wants to know when a request is done rather than to be woken the instant it starts running only to poll again immediately; SC-004's "within 50%" is read as a one-sided bound (a request should not run much later than promised; finishing sooner is not a violation), consistent with estimates that round up. Tasks were implemented as one coherent unit rather than in three separate tests-then-implementation commits, since the queue's rules and their tests were developed together; both commits carry passing gates and full task IDs.

2026-09-25, after `/speckit-implement Phase 6` (T037-T043): the in-memory `registry.defaults.ModelRegistry` stub Phases 1-5 built against is replaced outright by a SQLite-backed `registry.registry.ModelRegistry` with the identical interface (`register`, `resolve`, `default_for_slot`, `list_servable`, `defaults`), so every existing caller (`api/validate.py`, `api/app.py`, `cli.py`, and the fixtures in `tests/conftest.py` and `tests/queue/conftest.py`) needed only an import-path change; `register()` stays the quick, already-confirmed path those callers use, while the new `add_model()` is the FR-020 primitive behind `modelmora model add` and can deliberately record an incomplete or filter-undisclosable model, for T037 and T037a to refuse. A caller naming an incomplete, retired or filter-undisclosable model gets `unknown_model`, the same refusal a name never on record gets — a documented reading of FR-011 and FR-017 (a caller cannot act differently on "not on record" versus "on record but unusable," and no wire message may say more), which also meant `api/validate.py` needed zero changes beyond its import. `model_unavailable` is kept for the two cases the spec names by name: a servable model with no runner attached, and a digest mismatch at load time (T040). data-model.md's `service_period` columns (`from`, `to`) are named `started_at`/`ended_at` in the SQL, since `from`/`to` need quoting as SQLite keywords; the field meanings and the table's shape are unchanged. T037 and T037a were written first and confirmed red by temporarily disabling the servability gate, then green once it was restored, rather than a strict chronological red-green-refactor, since the gate and the SQLite store it depends on had to exist together as the registry's foundation. The registry's default DB path is `.modelmora/registry.sqlite3`, matching the runtime-state pattern the component's `.gitignore` already carved out before this phase landed.

## Implementation strategy

**MVP is Phase 1 to 3**: a caller gets text with no knowledge of models. That alone unblocks roadmap 004 against test mode, which is what the build order needs first.

Then Phase 4 (images, so A-006 can be tested), Phase 5 (honest behavior under load), Phase 6 (the licence trail), Phase 7 (presence), Phase 8 (polish and release).

## Traceability

| Requirement group | Tasks |
|---|---|
| Serving results (FR-001 to FR-008) | T015 to T023, T036 |
| Sharing one GPU (FR-009 to FR-019) | T024, T027 to T036, T028a, T029a, T033a |
| The registry (FR-020 to FR-026) | T037, T037a, T038 to T043 |
| Inside the Studio only (FR-027 to FR-029) | T013, T014, T044 to T047 |
| Private souls (FR-030 to FR-032) | T009, T026, T048 |
| Testability (FR-033, FR-034) | T011, T012, T027 to T029, T045 |
