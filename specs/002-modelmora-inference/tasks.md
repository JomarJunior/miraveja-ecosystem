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
- [x] T023 [US2] Implement image generation in `components/modelmora/src/modelmora/runners/image.py` using `diffusers`, honoring size, seed, steps, guidance and things to avoid
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

- [x] T044 [US5] Write `components/modelmora/tests/api/test_availability.py` first: `starting` rather than a failure before models are ready, `running` with the servable counts and the length of the line, nothing about another caller's requests, and — with an empty registry — servable counts of zero plus a request refused `model_unavailable`, never a failure of the service (FR-029, US5 scenarios 1 and 3, spec Edge Cases)
- [x] T045 [US5] Write `components/modelmora/tests/queue/test_shutdown.py` first: on stop, every waiting or running request ends `stopped_before_completion`, and zero requests are left without a final answer (FR-029, SC-009)
- [x] T046 [US5] Implement the lifecycle in `components/modelmora/src/modelmora/api/lifecycle.py`: the `starting`, `running` and `stopping` states, refusing new work with `starting` or `stopping` and a retry time, and draining the line on shutdown
- [x] T047 [US5] Implement the `availability` path in `components/modelmora/src/modelmora/api/availability.py`, reporting state, queue length and servable counts per kind (FR-029)

---

## Phase 8: Polish

- [x] T048 [P] Write `components/modelmora/tests/privacy/test_no_content_persisted.py`: after a run whose requests carry a unique marker phrase, a search of every log, error report and the registry finds it zero times; activity records hold only caller, model, settings, times and outcome (FR-030, SC-007)
- [x] T049 [P] Write `components/modelmora/src/modelmora/checks/studio_smoke.py`: the manual on-Studio check from quickstart Scenario 7 (real text and image models, a forced eviction, availability, shutdown), excluded from CI
- [x] T050 [P] Write `components/modelmora/docs/usage.md`: how a Studio component submits, polls, withdraws and collects; how a team member adds and retires a model; and why model names must never reach visitors (Principle IV, spec Assumptions)
- [x] T051 Walk every scenario in `specs/002-modelmora-inference/quickstart.md` on a clean checkout and correct anything that does not behave as written
- [ ] T052 Release `components/modelmora/` v1.0.0: tag `v1.0.0` and publish a GitHub release from the tag
- [ ] T053 Record in `docs/foundation/FOUNDATION-LOG.md` that spec 002 is implemented, and close it with a pointer to roadmap 003

---

## Phase 9: The Studio's own model collection (amendment, 2026-09-26)

**Goal**: the Studio serves its own existing model collection (`/data/models`), not models **🧠 ModelMora** downloads itself; T052 and T053 wait for this phase, not the other way around.

**Independent test**: `modelmora serve` (no `--test-mode`) answers a text request from the Studio's GGUF model and an image request from one of its SDXL checkpoints, with a natural eviction between them under the real GPU's own capacity — no capacity reduced by hand to force it.

- [x] T054 [P] Write `components/modelmora/tests/registry/test_local_path_migration.py` first: a database written before this amendment gains `local_path` and `companion_paths` without losing a record or its licence trail; reopening an already-migrated database is a no-op
- [x] T055 Amend FR-020 (and the `Model record` key entity) in `specs/002-modelmora-inference/spec.md`, and the `model` table in `data-model.md`: each record also holds where its files sit on the Studio (a local path, file or directory) and any companion files a runner needs beside them (a vision projector, a VAE)
- [x] T056 [P] Write `components/modelmora/tests/registry/test_runner_selection.py` first: given records of different kind and file format (`.gguf` text with a companion vision projector, a directory text model, a single-file image checkpoint, a directory image model), `build_runner` returns the matching runner class with the right upfront footprint hint, against tiny synthetic files — no GPU, no real weights
- [x] T057 Implement the schema migration (`components/modelmora/src/modelmora/registry/schema.sql`, `store.py`) and `ModelRegistry.add_model`/`list_servable_records` for `local_path`/`companion_paths`; implement `components/modelmora/src/modelmora/runners/build.py` (`build_runner`, `declared_footprint_hint`) and wire it into `cli.py`'s `_run_serve`, replacing the empty `runners` dict non-test-mode always built before
- [x] T058 Implement `components/modelmora/src/modelmora/runners/llamacpp.py`: `LlamaCppTextRunner` manages a `llama-server` subprocess over loopback HTTP (standard library only, so importing it never needs the `gpu` extra), reading images through a vision projector (`--mmproj`) when the binding and model support it
- [x] T059 Implement single-file loading in `components/modelmora/src/modelmora/runners/image.py`: `ImageRunner` picks `from_single_file` (`StableDiffusionXLPipeline`) or `from_pretrained` (`StableDiffusionPipeline`) by whether `local_path` names a file or a directory
- [x] T060 Register the Studio's own collection in `.modelmora/` (gitignored runtime state, never committed) through the CLI: the GGUF text model as the default for `text` and `text_with_images`, one SDXL-derived checkpoint as the default for `image`, the rest on record but not default; record each licence honestly, confirmed as the Visionary's, per the facts the Visionary is told separately
- [x] T061 Point `components/modelmora/src/modelmora/checks/studio_smoke.py` and quickstart Scenario 7 at the real `modelmora serve` path with these models (no artificially reduced capacity — the real 24GB GPU is what forces the eviction); run on the Studio and record real timings and VRAM

**Checkpoint**: `modelmora serve` builds a real runner from a registry record; the Studio's own collection, not a downloaded stand-in, is what roadmap 004 onward will actually talk to.

---

## Phase 10: Convergence

- [x] T062 Verify a model's files against its recorded digest before its first load in `modelmora serve` (not only in `modelmora model verify`), covering companion files such as a vision projector too; a mismatch refuses `model_unavailable` and logs the team's WARNING line, tested first against tiny synthetic files per FR-022, US4/AC3 (missing)
- [x] T063 Make GPU residency accounting match real use: take capacity from the actual GPU rather than a fixed 24 GiB, and count runtime overhead (SDXL activations at the requested size, the llama.cpp KV cache) in each runner's footprint, so a pair that would not truly fit is never loaded together per FR-009, US2/AC2 (partial)
- [x] T064 Load single-file SDXL checkpoints fully offline in `modelmora serve`: supply the pipeline config and tokenizer files locally (no Hub lookup at load time, no dependence on `HF_HOME` set only by `checks/studio_smoke.py`), and prove a load succeeds with networking disabled per FR-028 (partial)
- [x] T065 Refuse, before queueing, a text request whose conversation plus `maxLength` exceeds the `.gguf` model's context window (or size the context to the request), rather than accepting it and failing or truncating mid-generation per FR-007, FR-011 (partial)
- [x] T066 Declare the image runner's producible sizes (SDXL limits, width and height multiples of 8) through `max_image_dimensions` and the capability check, so an unproducible size is refused before queueing per US2/AC3 (partial)
- [x] T067 Make seeded requests to `LlamaCppTextRunner` reproducible (for example no prompt-cache reuse when a seed is given) and add a same-seed, same-output check to `checks/studio_smoke.py` per FR-006, US1/AC6 (partial)
- [x] T068 Stop returning a model's reasoning trace (`reasoning_content`) as the result in `runners/llamacpp.py`: disable thinking for requests, or end the request `failed_during_generation` with a reason when no answer was produced per FR-001, FR-007 (contradicts)
- [x] T069 Give each `LlamaCppTextRunner` its own free loopback port, confirm the answering `llama-server` is serving the expected model file before declaring it healthy, and make the subprocess die with its parent so a crash never leaves one holding the GPU per FR-007, FR-014 (partial)
- [x] T070 Validate `local_path` and every companion path on `model add` as existing, absolute paths on the Studio, rejecting URLs or Hub identifiers, tested first per FR-026, FR-028 (partial)
- [x] T071 Update `specs/002-modelmora-inference/plan.md` Summary and Technical Context to record the Phase 9 runners (`llama-server` for `.gguf` text, single-file SDXL) and the Studio-side `llama-server` binary requirement per plan: dependencies (partial)

---

## Phase 11: Convergence

- [x] T072 Report a model that cannot be loaded (runner startup error, missing binary, a pipeline that fails to load) as `model_unavailable` with the team's WARNING line, not `failed_during_generation`, and refuse further requests for that model before queueing until it loads again or `serve` restarts, tested first with a stand-in whose load fails per spec Edge Cases, FR-011, FR-022 (partial)
- [x] T073 Record the single-file SDXL pipeline config and tokenizer directory as a registry companion (`config`), covered by the companion digest check and set through `modelmora model add`, so `modelmora serve` loads image checkpoints offline from the registry alone, without `MODELMORA_SDXL_CONFIG_PATH` or `HF_HOME` in its environment per FR-028, FR-020, FR-025 (partial)
- [x] T074 Fail `LlamaCppTextRunner.load()` with a reason the team can act on when `llama-server` did not actually offload the model to the GPU (for example its CUDA runtime was not found), instead of silently serving from the CPU while residency counts GPU memory it does not use per FR-009, plan: GPU residency (partial)

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

2026-09-25, after `/speckit-implement` T023 and Phase 7-8 (T044-T051) on the Studio (RTX 4090, driver 595.84): `runners/image.py` mirrors `runners/text.py` exactly (lazy `torch`/`diffusers` imports, a footprint measured on `load()`). Proving it and US2 acceptance scenario 2 for real surfaced two defects no stand-in could show, both fixed here rather than deferred, since T049 could not otherwise pass: (1) `declared_footprint_bytes()` is asked before `load()` by both `Residency.ensure_loaded` and `check_image_capability` (FR-009, FR-011); a stand-in already knows its fake number then, but a real runner only learns its true footprint once weights are on the GPU, so it reported 0 and both checks passed trivially — eviction and the "too large for this Studio" refusal were silently defeated for any real model. `runners/text.py` and `runners/image.py` now take an optional `declared_footprint_bytes` construction hint that a caller who has already measured the model once (as `checks/studio_smoke.py` does before serving) supplies; `load()` still overwrites it with the measured figure, `unload()` restores the hint rather than dropping to zero. (2) `api/lifecycle.py`'s `shutdown()` originally signalled the worker thread without joining it, so a caller's `stopped_before_completion` answer was never delayed by a generation still finishing in the background; on the Studio, abandoning a thread mid-`torch` call this way aborted the whole process (SIGABRT) once the interpreter tore it down a moment later. The caller-visible answer is still set before any join, so it stays immediate; `shutdown()` now waits for the worker with its ordinary default timeout, which is what let `studio_smoke.py` exit 0 (twice in a row). `tests/queue/test_shutdown.py` releases its gate from a `threading.Timer` right after calling `shutdown()` so the join stays fast and deterministic rather than paying its full timeout. T051's walk found `quickstart.md` Scenario 4's `pytest -k "degrade or fidelity"` selected zero tests (`tests/queue/test_no_degradation.py` contains "degradation", not "degrade", and nothing anywhere is named "fidelity"); corrected to `-k no_degradation`. Scenario 7 was rewritten to match reality: `modelmora serve`'s own registry-to-runner wiring for the CLI remains a deliberate seam no task yet closes (a registry record has no field naming a loadable local weights path or device — `cli.py`'s `_run_serve` still builds an empty `runners` dict outside test mode), so `checks/studio_smoke.py` builds its own in-process service with real runners wired directly, the same way `_test_mode_registry` wires stand-ins, rather than talking to a separately started `serve` process; the quickstart no longer tells an operator to start one first. Chosen models, recorded honestly for this run: a small open text model (text, Apache-2.0, not gated) and a small open image model (image, an open model licence, not gated), ~1GB and ~3GB fp32 on disk respectively, both well under the GPU's 24GB and downloaded to `/data` rather than the Studio's near-full root filesystem (5.8GB free) — a machine-specific constraint, not a spec one. T052 (tag and release v1.0.0) and T053 (FOUNDATION-LOG entry, closing the spec) are deliberately left for the Visionary to confirm.

2026-09-26, after `/speckit-implement` Phase 9 (T054-T061) on the Studio: the Visionary directed that **🧠 ModelMora** serve the team's own existing model collection (`/data/models`) rather than models downloaded for the spec — the a small open text model/a small open image model stand-in models from the previous entry are retired from the registry's own advice, not from the code; nothing forces their removal, but they are no longer what `serve` defaults to. FR-020 and `data-model.md`'s `model` table are amended (T055): a record now carries `local_path` and `companion_paths`, closing the registry-to-runner seam the previous entry named. `registry/store.py` migrates an existing database (`ALTER TABLE ... ADD COLUMN`, since `CREATE TABLE IF NOT EXISTS` alone would not reach one already created) rather than requiring a fresh file. `runners/build.py` chooses the runner by kind and file format; `cli.py`'s `_run_serve` now builds a real runner for every servable record instead of the empty dict the previous entry left. Two new runner shapes: `runners/llamacpp.py` (`LlamaCppTextRunner`, a managed `llama-server` subprocess, standard library only) for the collection's `.gguf` text model, and single-file loading in `runners/image.py` (`from_single_file`, `StableDiffusionXLPipeline`) for the collection's SDXL checkpoints, chosen by whether `local_path` names a file or a directory.

Getting a real `.gguf` architecture ((its own GGUF architecture name), per its own GGUF metadata) running on this Studio took three attempts, in order: building `llama-cpp-python` with CUDA from source deadlocked the kernel mid-compile (a parallel `cc1plus` VMA-lock bug on kernel 7.0, unrelated to this repository) and was abandoned, not retried, per the Visionary's direction — four `cc1plus` processes were left unkillable in `D` state and the Studio needed a reboot; a prebuilt CUDA wheel from the project's own wheel index (`cu124`) loaded and ran, but crashed with `SIGILL` on this CPU (no AVX-512, which that wheel's CPU-fallback kernels assumed); the official the llama.cpp project GitHub release binaries (build b11191, dated the day of this run) shipped a per-microarchitecture CPU backend (an `alderlake` variant among others, correctly avoiding AVX-512) and worked immediately, offloaded to CUDA, with real multimodal support through `--mmproj` confirmed by asking it the color of a synthetic test image and getting the right answer. `LlamaCppTextRunner` therefore reads images through the vision projector when the registry names one as a `mmproj` companion (T058); nothing about this runner is a text-only fallback.

A second real defect surfaced only by running `modelmora serve` itself end to end (T061): `shutdown()` stopped the worker and drained the line but never told a resident runner to unload, leaving `llama-server` running and holding ~19GB of VRAM after the parent process had already exited cleanly -- an OS reaps a crashed process's own CUDA context automatically, but not a *child* process a runner started, which is exactly what `LlamaCppTextRunner` is. `shutdown()` now unloads every runner `Residency` still considers resident, right after the worker stops; re-running the check confirmed the GPU returns to baseline immediately, independently checked with `nvidia-smi`.

`declared_footprint_bytes()` for the two new runner shapes never needed the "measure once, then use the hint" dance the previous entry added for `transformers`/`diffusers`: a `.gguf` file's (and its `mmproj` companion's) size on disk is already a fair upfront estimate, known without loading anything, and `runners/build.py`'s `declared_footprint_hint` uses the same idea (a file's size, or a directory's summed) for every record built through it.

`compute_digest` (`registry/verify.py`) read an entire file into memory before hashing it; harmless for every fixture any test uses, but the collection's checkpoints run 7-18GB each, and reading one that size into RAM under real memory pressure (see below) risked exhausting it just to record a model. Changed to stream in 8MiB chunks; the digest is unchanged, proven by the existing registry tests passing without modification.

License review (for the Visionary; flags noted, not decided here): all seven registered models are non-gated, downloaded from nowhere new (T060 registers files already on the Studio). The GGUF text model, the Studio's GGUF text model, states `general.license: apache-2.0` in its own GGUF metadata (both the main file and its `mmproj`), naming `<model page>` as the base model; requantized by `a third party`. All six SDXL-architecture image checkpoints share one base license, the **the image checkpoints' shared base licence** (`<licence page>`), whose text states plainly: *"The output of this software is not covered by this license, and no contributor claims any rights to it"* -- outputs are unrestricted by this license; the license's own "Prohibited Uses" section governs using the *model* for law-violating, exploitative, discriminatory or similarly harmful purposes, not exhibiting art made with it. Each checkpoint's own the checkpoints' source site page additionally grants `allowCommercialUse` including `Image` (commercial use of generated images, specifically), which every one of the six has. Two flags for the Visionary: another image checkpoint is the checkpoints' source site-flagged `nsfw: true` at `nsfwLevel: 60`, the highest of the six, and its `allowCommercialUse` list is the narrowest (its image-use permissions, missing `Rent` and `Sell`) -- the Museum Charter permits explicit content when labeled, but this is worth a deliberate decision, not a default. one image checkpoint is the most restrictive of the six on the model's own reuse (`allowDerivatives: false`, `allowNoCredit: false`, `allowDifferentLicense: false`) and carries a training trigger word  noted in its record for completeness; neither restriction blocks serving it or exhibiting its outputs, but both are recorded honestly rather than smoothed over. the Studio's GGUF text model's Apache-2.0 is unambiguous.

Real Studio numbers (RTX 4090, driver 595.84, `modelmora serve` run as a real subprocess, no capacity reduced by hand): a text request to the Studio's GGUF text model in 4.0-4.2s; an image request to the default image checkpoint at 832x1216/24 steps in 15.8-20.6s, forcing a natural eviction of the text model (GPU 19167-19172 MiB while the text model was resident, down to 15219-15224 MiB once only the image model was); a second text request afterward reloaded the Studio's GGUF text model correctly (8.6-32.3s, including its own reload); a clean SIGINT shutdown exited 0 and returned the GPU to its ~450-460 MiB baseline both times run. `from_single_file`'s one network dependency was exercised for real: `the base SDXL pipeline`'s config and tokenizer files (no weights), 1.7MB total, cached once under `/data/hf-cache`. The `llama-server` binary and its CUDA runtime libraries (build b11191, ~591MB combined) were downloaded once from the llama.cpp project's GitHub releases to `/data/llamacpp/`, outside this repository, as the project's own prebuilt binaries -- a small binary, not a model download, per the Visionary's direction.

2026-09-28, after `/speckit-implement` Phase 10 (T062-T071) on the Studio: `verify_before_load` (registry/verify.py) had only ever been reachable from `modelmora model verify`, so `serve` itself never checked a model's files before loading them (T062). `VerifyingRunner` wraps every real runner `cli.py`'s `_run_serve` builds, checking a record's `local_path` and every companion file against their recorded digest once, lazily, before that runner's very first `load()` -- hashing a Studio-sized checkpoint takes real seconds, so this happens exactly once per process, not on every reload an idle timeout or an eviction causes. Companion files had no digest concept at all before this: a new `companion_digests` column (the same migration pattern as T054's `local_path`/`companion_paths`, `Store._ensure_companion_digest_column`) is backfilled automatically the first time an existing database is opened, computing a digest from whatever companion file is already on disk today -- proven against the live `.modelmora/registry.sqlite3`, all seven records and their licence trail intact afterward. `registry/digest.py` is a new module carrying `compute_digest` out of `verify.py`, so `registry/store.py` can call it during its own migration without an import cycle.

`worker/residency.py`'s `DEFAULT_CAPACITY_BYTES` (a fixed 24 GiB) is now only the stand-in/CI default; `detect_gpu_capacity_bytes()` asks `nvidia-smi` (falling back to `torch.cuda.mem_get_info`) for the real GPU's free memory, and `cli.py`'s non-test-mode `_run_serve` passes that in explicitly once at startup (T063). Footprints previously counted only resting weights, ignoring the memory a generation itself needs while it runs: `runners/image.py` gained `estimate_activation_overhead_bytes`, calibrated from the previous entry's own measured numbers (the default image checkpoint at 832x1216 measured ~15.2GB total against a ~7GB parameter footprint, so ~8.2GB of that is CFG-doubled UNet/VAE activations, scaled by pixel count); `runners/llamacpp.py` gained an analogous KV-cache calibration from the Studio's GGUF text model's own measured ~19.2GB against ~18.8GB of file sizes at context 4096. Both are documented as calibrated constants, not first-principles memory models (Principle IX). Since only the request about to actually run incurs this overhead -- an idle resident model holds only its weights -- `Residency.ensure_loaded` gained a `footprint_override` parameter used only for the request being admitted, computed once by `api/validate.py`'s `check_image_capability` (now also the T066 multiple-of-8 check) and carried on `QueuedRequest.footprint_hint`; every other resident runner's eviction math is unchanged.

T064's single-file SDXL loading fetched its pipeline config and tokenizer from the Hub at load time whenever `HF_HOME` was not already set by the caller (only `checks/studio_smoke.py` set it); `ImageRunner` now forces `HF_HUB_OFFLINE=1` and `local_files_only=True` unconditionally for a single-file load, and reads `MODELMORA_SDXL_CONFIG_PATH` for an explicit local snapshot directory that bypasses Hub repo-id resolution entirely. Proven offline on the Studio: loading `<image checkpoint>` with `config=` pointed at the already-cached `the base SDXL pipeline` snapshot succeeded in well under a second with no network reachable, since `HF_HUB_OFFLINE` makes `huggingface_hub` raise rather than silently fall back to the network -- a successful load under that flag is itself the proof nothing reached it.

T065 refuses a text request before queueing when its conversation plus the requested output length could not fit a `.gguf` runner's context window (a new `context_window_tokens()` on `Runner`, `None` for the stand-ins and `TextRunner`, the configured `-c` value for `LlamaCppTextRunner`), estimated with a deliberately generous chars-per-token heuristic that never refuses something that would actually have fit.

T067's investigation found this exact Studio, model and build (b11191) already reproducible for a same-seed, same-request pair even with `llama-server`'s prompt cache left on; the fix (`cache_prompt: false` whenever a seed is given) was made anyway, since the promise (FR-006) must not depend on that being true of every model and build, and it costs nothing when there is nothing to reuse. T068's fix turned out simpler than the task text's own suggested options: build b11191 ships a server-level `--reasoning off` flag, confirmed against the real model to make `content` always populated and `reasoning_content` never appear at all, rather than a per-request `chat_template_kwargs` workaround. Both were checked together against a real `llama-server` instance with its CUDA runtime correctly on `LD_LIBRARY_PATH` -- an earlier manual check without it silently fell back to CPU inference (473 MiB GPU, ~3.3 tokens/sec) and would have "confirmed" both fixes against results that were never actually running on the GPU at all; the corrected check (19152 MiB, ~38 tokens/sec) is what the numbers above are drawn from.

T069's dynamic port allocation binds a throwaway loopback socket to port 0 to find one free, closes it, and hands that port to `llama-server`; an explicit constructor argument or `MODELMORA_LLAMA_SERVER_PORT` still wins when given, so a single pinned instance (or a test) is unaffected. The identity check (`/props`'s `model_path`, an exact match against the `-m` argument the runner started with) runs immediately after `/health` first succeeds, inside the same startup wait, so a stale server already listening on that port fails `load()` outright rather than being declared healthy. `_die_with_parent` (`PR_SET_PDEATHSIG` via `ctypes`, no new dependency) is passed as `subprocess.Popen`'s `preexec_fn`, so a `llama-server` child is sent SIGTERM by the kernel itself if this process is ever killed outright rather than reaching `unload()`'s own `terminate()`.

T070's path validation (`ModelRegistry.add_model`, so every caller gets it, not only the CLI) rejects a URL (a literal `://` check) or a relative path (including a bare Hub identifier, which is simply never absolute) before anything is written, and a nonexistent absolute path the same way; none of the seven already-registered live records needed re-adding; this only guards what is added from here on.

Real Studio numbers for this phase (RTX 4090, driver 595.84, `modelmora serve` run as a real subprocess with capacity now auto-detected from the live GPU, not hand-tuned): a text request to the Studio's GGUF text model in 11.7s, an image request to the default image checkpoint at 832x1216/24 steps in 23.1s forcing a natural eviction (19187 MiB resident with the text model, down to 15227 MiB with only the image model resident, both consistent with the previous entry's numbers), a second text request reloading correctly in 5.3s, an identical same-seed same-request text result checked twice, and a clean SIGINT shutdown returning the GPU to its ~454-485 MiB baseline every run (checked twice in a row). The live `.modelmora/registry.sqlite3` was backed up before this run and diffed afterward: all seven records, their licence trail and their default-slot assignments unchanged, with `companion_digests` newly populated for the Studio's GGUF text model's `mmproj`. No model weights and no new binaries were downloaded; the `llama-server` binary and its CUDA runtime already present from the previous entry were reused as-is.

2026-09-28, after the second `/speckit-converge` (Phase 11, T072-T074), implemented in the main session: a model whose load fails for any reason other than a digest mismatch was reported `failed_during_generation` with only an exception name, the team was not told, and every later request for it was accepted only to retry the same failed load. `Residency.ensure_loaded` now records a failed load, ends that request `model_unavailable`, and logs one WARNING line with the model and the error's own reason; `api/validate.py`'s `check_loadable` refuses later requests for that model before queueing until `serve` restarts. Logging the load error's message is safe under FR-030 because `load()` is never handed a request. T073 closes the last environment dependency T064 left: the single-file SDXL pipeline config and tokenizer directory is now a `config` companion on each image record, digested and checked before the first load (T062), attached to existing records with a new `modelmora model add-companion` (a recorded role is never replaced; a changed model is a new version), and passed to `ImageRunner` by `runners/build.py`. The directory is a dereferenced copy of the cached `the base SDXL pipeline` config snapshot (1.7MB cached, 3.2MB copied, no weights, nothing fetched) at `/data/modelmora-assets/sdxl-base-1.0-pipeline-config`, outside the Hub cache so pruning that cache cannot break a load; all six image records gained it, with their licences and the defaults unchanged. `MODELMORA_SDXL_CONFIG_PATH` stays only as a fallback for a record without one. T074: once `llama-server` is healthy, `LlamaCppTextRunner` asks `nvidia-smi` how much GPU memory that server process holds; under half the weights' size means it fell back to the CPU, so the server is stopped and the load fails naming `MODELMORA_LLAMA_CUDART_LIB_DIR`. With no GPU to ask (CI), the check cannot tell and does not block. Verified on the Studio: the smoke check passed with neither `HF_HOME` nor `MODELMORA_SDXL_CONFIG_PATH` set, and `~/.cache/huggingface` was never created (text 12.7s at 19202 MiB, image 20.6s at 15253 MiB after a natural eviction, reload 9.0s, same-seed text identical, clean shutdown back to 486 MiB). Starting the Studio's GGUF text model without its CUDA runtime was refused `model_unavailable` in 2.7s, with the WARNING line reading "0 MiB used for 17977 MiB of weights", and no `llama-server` left running. 162 tests pass with no GPU.

## Implementation strategy

**MVP is Phase 1 to 3**: a caller gets text with no knowledge of models. That alone unblocks roadmap 004 against test mode, which is what the build order needs first.

Then Phase 4 (images, so A-006 can be tested), Phase 5 (honest behavior under load), Phase 6 (the licence trail), Phase 7 (presence), Phase 8 (polish and release).

## Traceability

| Requirement group | Tasks |
|---|---|
| Serving results (FR-001 to FR-008) | T015 to T023, T036 |
| Sharing one GPU (FR-009 to FR-019) | T024, T027 to T036, T028a, T029a, T033a |
| The registry (FR-020 to FR-026) | T037, T037a, T038 to T043, T054 to T060 |
| Inside the Studio only (FR-027 to FR-029) | T013, T014, T044 to T047 |
| Private souls (FR-030 to FR-032) | T009, T026, T048 |
| Testability (FR-033, FR-034) | T011, T012, T027 to T029, T045 |
