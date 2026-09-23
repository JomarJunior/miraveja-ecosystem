# Implementation Plan: Model Inference in the Studio

**Branch**: `002-modelmora-inference` | **Date**: 2026-09-23 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/002-modelmora-inference/spec.md`

## Summary

**🧠 ModelMora** is one long-running process on the Studio machine that owns the GPU. Callers reach it over **loopback HTTP** and never touch a model; they submit a request, get an immediate acceptance with a position and an estimate, then collect the result. Inside, a **single-threaded generation worker** loads and unloads open-weight models with `transformers` and `diffusers`, so exactly one thing uses the GPU at a time and residency is decided in one place. A **SQLite registry** on the Studio holds every model's name, version, kind, license and service dates, retired records included. A **test mode** with tiny stand-in models lets **🎭 SonaVida**, **💬 DescriDiva** and **🧐 CuraGusta** be built and tested with no GPU.

## Technical Context

**Language/Version**: Python 3.12 (same as the Studio's other components).

**Primary Dependencies**: an ASGI framework for the loopback API; `transformers` for text (including image-reading text models); `diffusers` for image generation; `torch` with CUDA; `sqlite3` from the standard library; `pydantic` v2 for request and result models; `pytest` for tests. Exact versions pinned in `/speckit-tasks`.

**Storage**: one SQLite file for the registry and the served-model history. Generated results are held in memory, with images spilled to a temporary directory, and discarded after the holding time (FR-032). Request and result content is never written to any durable store (FR-030).

**Testing**: `pytest`, entirely without a GPU via the test mode (FR-033, FR-034). A separate, manually run suite exercises real models on the Studio.

**Target Platform**: one Linux or macOS machine with an RTX 4090 (24 GB). The service binds `127.0.0.1` only.

**Project Type**: a local service plus a CLI, in `components/modelmora/`.

**Performance Goals**: first answer within 1 second while the GPU is busy (SC-003); 90% of requests start within 50% of the estimate given at acceptance (SC-004); a burst of 20 mixed requests from 4 callers all resolve (SC-002).

**Constraints**: one GPU, one process, no added cost (Principle IX); no network reachability beyond loopback (FR-027) and no network at all once models are local (FR-028); never degrade a request to relieve load (FR-007); no content judgment (FR-008).

**Defaults** (pinned here rather than left to implementation): line limit 32, never below the SC-002 burst size; bounded overtaking 2 minutes; result holding 1 hour; idle-unload 10 minutes.

**Telling the team**: where the spec says the team is told — a version mismatch, a model that cannot be loaded — the channel is one `WARNING` line in the operator log naming the model and version, plus a non-zero exit from `modelmora model verify`. The caller separately gets `model_unavailable`. Nothing about a model's failure ever reaches a persona.

**Scale/Scope**: a handful of resident personas plus a curator, a few dozen requests an hour, single-digit models on record.

## Constitution Check

*GATE: passed before Phase 0, re-checked after Phase 1 design.*

| Principle | How this plan complies |
|---|---|
| **I. Personas Are Free Within the Charter** | No quotas, priorities or per-caller limits. Ordering is arrival order with one bounded exception that depends on the model, never on who asked (FR-019). A caller's chosen model is served as named or refused, never silently swapped (FR-004, FR-007). Studio limits surface as honest states a persona can act on. |
| **II. Memory, Never Metrics** | **🧠 ModelMora** never touches visitor activity. Queue position and estimates are operational facts for the caller, not audience signals, and never reach a persona as anything to optimize. |
| **III. Two Gates Before Exhibition** | No content judgment here (FR-008). A model's own built-in filter is disclosed rather than hidden, so the gates judge the real output. |
| **IV. Openly AI, Never Out of Character** | Availability states (`starting`, `running`, `stopping`) exist so **🎭 SonaVida** can render them as presence. Model names stay inside the Studio. |
| **V. The Studio Stays Behind the Door** | Binds loopback only; the registry cannot hold a hosted model; works offline once models are local. |
| **VI. One Contract Between Worlds** | Out of scope by design: **🧠 ModelMora** is not on the Studio Link and never speaks to the Museum side. Its local API is versioned all the same, since three components will depend on it. |
| **VII. Quality Over Speed** | Tests precede implementation for the queue, the registry gate and refusal reasons. A request is never quietly downgraded to go faster. |
| **VIII. Open Code, Private Souls** | Persona prompts pass through memory only: no request or result content in logs, errors or the registry (FR-030), proven by the marker search in SC-007. |
| **IX. Frugal by Design** | One process, one SQLite file, no broker, no second machine, no paid service. Idle models are unloaded to keep the GPU free. |

**Post-design re-check (after Phase 1, revised 2026-09-23 following `/speckit-analyze`):** passing. The analysis found one ordering slip against Principle VII — caller isolation is access control, so its test now precedes its implementation (T034 before T035) — and one gap the spec implied but nothing carried: a model whose built-in filter cannot be disclosed must never be servable, now a recorded judgment in the registry beside the licence. Two further design points were checked closely. First, the estimate must not become a lever: it is measured from this Studio's own history and is never used to reorder work (R-5). Second, the holding store for finished results is the one place content outlives a request, so it is memory plus a temporary directory wiped on shutdown, never the SQLite file (R-7).

**Violations:** none.

## Project Structure

### Documentation (this feature)

```text
specs/002-modelmora-inference/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output: the local API
│   └── modelmora-v1.yaml
└── tasks.md             # /speckit-tasks output, not created here
```

### Source Code

```text
components/modelmora/                    public repo, Apache-2.0
├── src/modelmora/
│   ├── api/            the loopback HTTP surface: submit, ask, withdraw, list, availability
│   ├── queue/          the line: admission, ordering with bounded overtaking, states
│   ├── worker/         the single GPU worker: residency, load, unload, generate
│   ├── runners/        text (transformers), image (diffusers), and the test-mode stand-ins
│   ├── registry/       SQLite records, licence gate, file verification, retirement
│   └── cli.py          serve, model add/list/retire, set-default
└── tests/
    ├── queue/          ordering, bounded overtaking, withdrawal, busy, shutdown — no GPU
    ├── registry/       incomplete records refused, retirement keeps history, version mismatch
    ├── api/            first answer under a second, states, caller isolation
    └── privacy/        the marker search behind SC-007
```

**Structure Decision**: one component repository, `JomarJunior/modelmora`, which already exists and is checked out at `components/modelmora/`. The split above follows the spec's own seams — serving, sharing one GPU, the registry — so a reader can find the rule and the code that keeps it in the same place. The GPU work sits behind `runners/` so the whole queue and API can be tested with stand-ins, which is what FR-033 and FR-034 demand.

## Complexity Tracking

No constitution violations, so nothing to justify.
