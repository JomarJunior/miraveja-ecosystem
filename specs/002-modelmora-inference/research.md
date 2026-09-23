# Phase 0 Research: Model Inference in the Studio

Decisions behind the plan. Those marked *(Visionary)* were chosen directly.

## R-1: How callers reach ModelMora *(Visionary)*

- **Decision**: one long-running process exposing an HTTP API bound to `127.0.0.1`.
- **Rationale**: three separate components will call it, and exactly one place must decide what sits on the GPU (FR-005, FR-009). Loopback satisfies "this machine only" (FR-027) with no extra infrastructure, and matches the request/response style already used for the Studio Link, so the Studio has one idiom rather than two.
- **Alternatives considered**: a Unix domain socket (marginally stronger isolation and free caller identification by file permissions, but poorer tooling; kept as a later hardening step); an in-process library (two awake personas would each load models and fight over the GPU, breaking FR-005 and FR-009).

## R-2: What runs the models *(Visionary)*

- **Decision**: `transformers` for text and image-reading text, `diffusers` for image generation, loaded and unloaded by **🧠 ModelMora** itself inside its own process.
- **Rationale**: FR-009 requires serving more models than fit at once with no caller seeing a memory error, which needs direct control of residency. Owning the load path also makes FR-022 (files must match the recorded version) and FR-008 (disclose a model's built-in filter) straightforward rather than second-hand.
- **Alternatives considered**: delegating to local model servers (far less code, but residency, eviction and version verification become whatever those servers allow); a split by kind (two mechanisms to reason about for one queue).

## R-3: The registry *(Visionary)*

- **Decision**: SQLite on the Studio, with tables for model records and service periods, reached through a CLI.
- **Rationale**: FR-023 needs retired records kept with their service dates, and FR-025 needs a team member to add, retire and change defaults without touching code. A small relational store gives both, plus the "list every model ever served with its license" review in SC-006.
- **Alternatives considered**: a YAML file (pleasant to edit and diff, but service history in a hand-edited file rots); YAML plus a history table (two stores to keep in step for no gain at this size).

## R-4: One GPU, one worker

- **Decision**: a single generation worker thread with an explicit residency policy. The API accepts requests immediately; the worker takes one at a time, loads what it needs, evicts least-recently-used models when memory is short, and unloads a model idle beyond a timeout.
- **Rationale**: the 4090's memory is the scarce resource, and concurrent generation on one GPU mostly trades throughput for latency and out-of-memory risk. One worker makes FR-009 achievable and the queue's behavior predictable enough for honest estimates.
- **Alternatives considered**: concurrent generation with a memory budget (higher utilization, but eviction races and memory errors are exactly what FR-009 forbids callers from seeing); a process per model (clean isolation, far too much memory for one machine, Principle IX).

## R-5: Ordering and estimates

- **Decision**: arrival order, with one exception: a request whose model is already resident may run ahead of an older request needing a load, until the older one has waited its bounded overtaking time (default 2 minutes), after which it runs next. Estimates come from measured history on this Studio: a rolling median of generation time per model and kind, plus load time when the model is not resident, plus the work ahead in the line.
- **Rationale**: FR-019 in one sentence. The exception is about the model, never the caller, so no persona is favored (Principle I). Estimates measured rather than declared is what SC-004 checks; nothing may reorder work to make an estimate look better.
- **Alternatives considered**: strict FIFO (thrashes the GPU when two models alternate); shortest-job-first (favors text over images, which favors some personas over others).

## R-6: Immediate acceptance

- **Decision**: submitting returns at once with a request id, the position in line and an estimate; the caller then polls its request, or waits on a long poll of up to 30 seconds. Admission is refused as *busy* when the line is at its limit.
- **Rationale**: SC-003 wants a first answer within a second even while the GPU is saturated, which rules out doing any generation on the request path. Polling keeps callers simple and matches how **🎭 SonaVida** already works over the Studio Link.
- **Alternatives considered**: blocking until done (a caller would hold a connection for minutes and learn nothing); server-sent events (streaming is out of scope, and this adds a second transport style).

## R-7: Where results live for their holding hour

- **Decision**: finished results are held in memory, with image bytes written to a temporary directory that is wiped on startup and shutdown, and discarded after the holding time (default 1 hour).
- **Rationale**: FR-030 keeps content out of anything durable; FR-032 needs a finished result to survive until its caller collects it. A temporary directory is the smallest thing that does both, and holding a few dozen images in RAM would be wasteful on a machine whose memory is the constraint.
- **Alternatives considered**: keeping images in memory only (simpler, but a burst of large images competes with model weights); writing to the SQLite file (durable storage of persona output — forbidden by FR-030).

## R-8: Verifying a model's files (FR-022)

- **Decision**: each record stores a digest over the model's weight files; the worker verifies it before the first load and refuses to serve on a mismatch, reporting to the team.
- **Rationale**: FR-022 exists so a result's recorded name and version really identify what produced it, which is what makes SC-005's license trail meaningful.
- **Alternatives considered**: trusting the directory name (a silent swap would break the license trail); re-verifying on every load (slow for multi-gigabyte weights with no added safety within one run).

## R-9: Identifying callers on the Studio machine (FR-017)

- **Decision**: each caller is issued a local token recorded in the Studio's configuration; it names the caller (for example `sonavida`) so a request can be shown only to its owner. This is not a security boundary — loopback is — but an ownership marker.
- **Rationale**: FR-017 requires a caller to see only its own requests; something must distinguish callers on a machine where they are all trusted.
- **Alternatives considered**: peer credentials over a Unix socket (better, and the reason the socket stays on the table); no identification at all (every caller could withdraw another's request by guessing an id).

## R-10: Test mode (FR-033, FR-034)

- **Decision**: `runners/` ships stand-ins that return deterministic text and a tiny generated image after a configurable delay, selected by configuration. They fake load time and memory footprint, so eviction, bounded overtaking, busy refusals and shutdown are all exercised without a GPU.
- **Rationale**: roadmap 004 to 006 must be buildable while the GPU is busy or the machine is off, and the queue rules are where the subtle bugs live. Fake memory footprints are what let eviction be tested at all in CI.
- **Alternatives considered**: tiny real models (slow in CI, and hardware-dependent); mocking at the API boundary (would leave the queue and residency logic untested, which is most of this feature).
