# Implementation Plan: Perception for Personas and the AI Gate

**Branch**: `005-descridiva-perception` | **Date**: 2026-09-29 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/005-descridiva-perception/spec.md`

## Summary

**💬 DescriDiva** is one small service on the Studio, bound to loopback, that turns one image into a neutral description, and into Charter notes for callers with the gate role (R-1, R-2). Each description is a single request to **🧠 ModelMora**'s image-reading text model, with a fixed seeing brief, temperature 0 and a fixed seed. The model answers with a structured **seen answer** (R-3). From there on everything is deterministic code (R-4):

- validation;
- raise-only reconciliation of the notes from the facts (R-5);
- a fixed-text protected case whenever a possible minor appears with sexual, suggestive or harmful content (R-6);
- a neutrality scan (R-7);
- rendering in a fixed seven-aspect layout;
- fitting within the Studio Link's 4,000 characters (R-8).

An unusable answer gets one deterministic retry, and after that the request ends *incomplete*. **💬 DescriDiva** keeps nothing on disk. In **🎭 SonaVida**, the perception seam's return is widened so a failed look is lived as spec 004 FR-029 already requires, closing a gap where the stand-in disguised failures as descriptions (R-11).

## Technical Context

**Language/Version**: Python 3.12, the same as the Studio's other components.

**Primary Dependencies**: `starlette` and `uvicorn` (loopback API, as in **🧠 ModelMora**), `httpx` (the **🧠 ModelMora** client, written against its v1 contract), `pydantic` v2 (API models and the seen answer), `Pillow` (image checks only), `pytest` and `pytest-asyncio`. Managed with `uv`. Exact versions pinned in `/speckit-tasks`. `miraveja-di` and `miraveja-authentication` were considered and are not needed (R-2, R-12).

**Storage**: none for content. `$DESCRIDIVA_HOME/callers.json` holds caller names, roles and token hashes. Results are held in memory until collected, or for 10 minutes (R-10). The evaluation set and its reports live on the Studio under `$DESCRIDIVA_HOME/`, outside every repository (R-13).

**Testing**: `pytest`, with no GPU and no network, against a scripted **🧠 ModelMora** transport (FR-033). Fixture images are drawn by code and show no person (FR-032). The minors rule is tested with scripted text only (FR-031). On the Studio, `descridiva evaluate` runs the evaluation set against the real **🧠 ModelMora** for SC-001 to SC-003 and SC-007.

**Target Platform**: the Studio machine (Linux, RTX 4090), next to **🧠 ModelMora** on loopback. Never reachable from the Museum side.

**Project Type**: a local service plus a CLI, in `components/descridiva/`. There is also a small fix at the perception seam in `components/sonavida/`, closing a spec 004 FR-029 gap (R-11).

**Performance Goals**: one **🧠 ModelMora** request per description, or two with the retry. **💬 DescriDiva**'s own work is under 50 ms per description, apart from waiting on the model. A description is ready within 2 minutes with nothing else waiting, including model loading (SC-006).

**Constraints**:

- binds `127.0.0.1` only (FR-026);
- sends the image unchanged, and never shortens or lowers work to relieve load (FR-023);
- keeps no content on disk or in logs (FR-028);
- notes go to gate callers only (FR-019);
- takes nothing from the caller but the image (FR-004);
- the same image and model give the same result (FR-005).

**Scale/Scope**: the pre-alpha roster's attempts plus the gate's candidates, a few dozen descriptions an hour at most. There are two callers: **🎭 SonaVida** now, and **🧐 CuraGusta** from roadmap 006.

## Constitution Check

*GATE: passed before Phase 0, re-checked after Phase 1 design.*

| Principle | How this plan complies |
|---|---|
| **I. Personas Are Free Within the Charter** | Descriptions carry no advice or judgment that could steer a persona (FR-008). Personas get no Charter notes, so they never work against the gate's warnings (FR-019). A failed look reaches the persona as a situation it chooses how to answer. **🎭 SonaVida** never retries on its behalf (R-11). **💬 DescriDiva** never lowers what it does under load (R-9, R-10). |
| **II. Memory, Never Metrics** | No score, rating or rank in any description or note (FR-008); the neutrality scan rejects numeric certainty (R-7). The evaluation's recall numbers are team-facing only and never reach a persona. |
| **III. Two Gates Before Exhibition** | **💬 DescriDiva** decides nothing and publishes nothing (FR-009, FR-030). Notes are raised on doubt and raised by code from facts, never lowered (R-5). A possible minor with sexual, suggestive or harmful content yields fixed text only (R-6). Access control on notes is tested first. |
| **IV. Openly AI, Never Out of Character** | Descriptions name no model, prompt or internal (FR-010, R-7). Provenance travels beside the text, never in it (FR-029). **🎭 SonaVida** translates every failure into the persona's life (R-11). |
| **V. The Studio Stays Behind the Door** | Loopback only, and the service refuses any other bind (FR-026). Open-weight models only, reached through **🧠 ModelMora** (FR-027). |
| **VI. One Contract Between Worlds** | Not on the Studio Link. The neutral description is shaped to fit the Studio Link's existing field unchanged (spec 001 FR-012), so no contract change is needed. **💬 DescriDiva** owns no one else's data and reads no other component's storage. |
| **VII. Quality Over Speed** | Tests before implementation for the access control on notes, reconciliation, the protected case, the neutrality scan and honest endings. |
| **VIII. Open Code, Private Souls** | No persona definition is ever involved (FR-004). Fixtures are drawn by code and show no person. The evaluation set stays on the Studio (FR-032), and `evaluate` refuses a set inside a git tree. The repository already runs the `miraveja-persona` guard. |
| **IX. Frugal by Design** | One small process, no database, no new model, one model request per image (R-3). |

**Post-design re-check (after Phase 1):** passing. Two points were checked closely. First, FR-019 is enforced, not conventional: the role check happens before any model request, and the result schema omits notes for viewers (R-2, contracts/descridiva-v1.yaml). Second, FR-017 holds whatever the model writes: the protected case is decided from structured facts and replaces free text with fixed text (R-6). Both rules are testable with scripted answers alone.

**Violations:** none. Two judgment calls are recorded under Complexity Tracking.

## Project Structure

### Documentation (this feature)

```text
specs/005-descridiva-perception/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   ├── descridiva-v1.yaml   # the loopback API
│   ├── seeing-brief.md      # what the model is asked and must answer
│   ├── cli.md               # the descridiva command line
│   └── perception-port.md   # SonaVida's Perception port: return widened, per spec 004 FR-029
└── tasks.md             # /speckit-tasks output, not created here
```

### Source Code

```text
components/descridiva/                    public repo, Apache-2.0 (exists; guard in CI)
├── src/descridiva/
│   ├── api.py            loopback app: ask, collect, withdraw, availability (contracts/descridiva-v1.yaml)
│   ├── callers.py        callers file, token hashes, roles (R-2)
│   ├── images.py         format, readability and single-frame checks (R-9)
│   ├── modelmora.py      loopback client to ModelMora v1, outcome types (R-10)
│   ├── brief.py          the seeing brief, version 1, and the retry correction (R-3, R-4)
│   ├── seen.py           the seen answer model and its validation (data-model.md)
│   ├── reconcile.py      raise-only notes and the protected case (R-5, R-6)
│   ├── neutrality.py     the scan and its word lists (R-7)
│   ├── render.py         the fixed layout and fitting to 4,000 characters (R-8)
│   ├── describe.py       one description end to end, pure except for the model call
│   ├── requests.py       in-memory requests, states, holding, in-flight cap (R-10)
│   ├── evaluate.py       the team's evaluation run and report (R-13)
│   └── cli.py            serve, callers, describe, evaluate
└── tests/
    ├── fixtures/         images drawn by code; scripted seen answers (no person in any image)
    ├── unit/             seen, reconcile, protected case, neutrality, render, fit, images
    ├── api/              gate-only notes, ownership, refusals, withdrawal, loopback bind
    ├── integration/      end to end against the scripted ModelMora: retry, filter, busy, stop
    └── privacy/          no content in logs, nothing on disk

components/sonavida/                      perception seam fix only (R-11)
├── src/sonavida/ports/perception.py      widened return and DescriDivaPerception
├── src/sonavida/standins/perception.py   outcomes instead of a sentence
├── src/sonavida/actions/creation.py      a failed look is not stored as seen
├── src/sonavida/life.py                  a failed look at submission: stays submitted, brought to the gate next turn
├── src/sonavida/translate.py             "the attempt could not be seen clearly"
├── src/sonavida/cli.py                   dry run uses DescriDiva when DESCRIDIVA_TOKEN is set
└── tests/                                existing tests unchanged; new tests for failed looks
```

**Structure Decision**: the component repository `JomarJunior/descridiva` already exists at `components/descridiva/`, with Apache-2.0 and the guard workflow. This feature adds its first code, a README headed **💬 DescriDiva** linking to the hub, and a test workflow. Modules follow the pipeline in [contracts/seeing-brief.md](./contracts/seeing-brief.md), so each rule the spec sets lives in one file with its own tests. Commits in both repositories reference spec 005.

## Decided during planning

- **A submission whose look failed** *(Visionary, R-11)*: the piece stays `submitted`, since spec 004's piece states only move forward. At the persona's next turn, **🎭 SonaVida** looks again and brings it to the gate. This completes the persona's own choice to submit, unchanged, rather than retrying a request it did not choose (spec 004 FR-029). Each look that fails again is remembered as its situation. Roadmap 006 may later move the look at candidates into **🧐 CuraGusta**.

## Dependencies outside this spec

- **SC-009's dry run** needs a resident persona frozen in **🔐 CofreAlma** (spec 004 contracts/cli.md). Until then it waits, as **🎭 SonaVida**'s real week does. No spec change is needed: SC-009 holds as written (R-11).

## Hub documents to update when the spec closes

- `docs/foundation/GLOSSARY.md`: **Neutral description**, **Charter note** (spec Assumptions).
- `specs/004-sonavida-persona-life/contracts/ports.md`: point the `Perception` row to [contracts/perception-port.md](./contracts/perception-port.md).
- `docs/components/descridiva.md`: the service, its port and roles.
- `docs/foundation/FOUNDATION-LOG.md`: this spec's decisions.

## Complexity Tracking

| Judgment | Principle or rule it touches | Why it is acceptable |
|---|---|---|
| `descridiva evaluate` reads Charter notes and keeps descriptions in a report on disk | FR-019 (notes to the gate only), FR-028 (keep nothing) | Both rules protect personas and the service's callers. The evaluation is the team checking the gate's eyes on its own synthetic set, which SC-001 to SC-003 require. It runs in process, not through the API. Its reports stay on the Studio under `$DESCRIDIVA_HOME`, and it refuses a set inside a git tree. No persona ever reads them. |
| A sentence that still breaches neutrality after the retry is removed rather than failing the whole description | FR-001 (every aspect answered) | Losing one sentence keeps an otherwise good, neutral description. If removal empties an aspect, the request ends *incomplete*, so FR-001 and FR-024 still hold. |
