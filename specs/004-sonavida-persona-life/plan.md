# Implementation Plan: A Persona's Life in the Studio

**Branch**: `004-sonavida-persona-life` | **Date**: 2026-09-25 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/004-sonavida-persona-life/spec.md`

## Summary

**🎭 SonaVida** is one long-running process on the Studio that hosts every living persona as its own task. A persona lives in **turns**: at each turn the runtime gives it who it is, where it is, what it is working on, what it recalls, and every action valid right now, each annotated with its own habits; a text model through **🧠 ModelMora** chooses one action, gives the reason in the persona's words, and says when it wants its next turn (hybrid turns, R-1). Memory is one append-only SQLite file per persona with full-text recall (R-2). The definition is read once, at birth, and copied into the persona's own `self` record (R-3). Perception and the AI gate are ports with stand-ins until **💬 DescriDiva** and **🧐 CuraGusta** exist (R-9); the Studio Link is `miraveja-studiolink` against the Museum side or its reference stand-in.

## Technical Context

**Language/Version**: Python 3.12, the same as the Studio's other components.

**Primary Dependencies**: `miraveja-persona` (loader, shared-past listing), `miraveja-studiolink` (Studio end client, reference stand-in for tests), `httpx` (the **🧠 ModelMora** loopback client, written here against its v1 contract), `pydantic` v2 (turn replies, ports), `sqlite3` with FTS5 from the standard library, `anyio`/asyncio, `pytest`. Managed with `uv`, like `modelmora` and `miraveja-studiolink`. Exact versions pinned in `/speckit-tasks`.

**Storage**: `$SONAVIDA_HOME/personas/<persona-id>/` per persona: `memory.sqlite` (self, entries, pieces, attempts, inbox; FTS5 index), images of attempts, and a `lock` file. Never inside a repository; nothing leaves the Studio (FR-034).

**Testing**: `pytest`, entirely GPU-free and network-free: the **🧠 ModelMora** stand-in, perception and gate stand-ins, and the `miraveja-studiolink` reference stand-in in process, with a simulated seeded clock so a week runs in minutes (FR-038, FR-039). Synthetic personas only (FR-037). A manual run against the real **🧠 ModelMora** on the Studio proves SC-001 and SC-002.

**Target Platform**: the Studio machine (Linux or macOS with the RTX 4090); **🧠 ModelMora** on its loopback; the Museum side over HTTPS through the Studio Link, Studio-initiated only.

**Project Type**: a local long-running service plus a CLI, in `components/sonavida/`.

**Performance Goals**: a turn costs one short text request, plus image and perception requests only when the persona makes an attempt. The runtime adds no noticeable overhead beyond **🧠 ModelMora**'s own waits. A simulated week for three personas completes in under 5 minutes in CI.

**Constraints**: no network listener (US5); Studio-initiated Studio Link only (Principle V); the runtime never lowers, retries or replaces a persona's request (FR-029); no counts, totals or ranks produced anywhere persona-facing (FR-027); no Studio internals in memory (FR-031); proposals never remove a valid action (R-1).

**Scale/Scope**: the pre-alpha roster of about 15 personas, each taking a turn every few minutes to hours by its own choice; memory grows by tens to hundreds of entries a day per persona.

## Constitution Check

*GATE: passed before Phase 0, re-checked after Phase 1 design.*

| Principle | How this plan complies |
|---|---|
| **I. Personas Are Free Within the Charter** | The persona chooses every action and its next turn; proposals always include every action valid in the state, including resting, doing nothing, abandoning and leaving (R-1, tested). Hints come only from the persona's own tendencies and memory. No timetable, quota, minimum or maximum anywhere (FR-005). Studio limits reach the persona as situations it chooses how to answer (R-7). The two-turn confirmation for leaving (R-11) guards against a garbled reply, not against intent; see Complexity Tracking. |
| **II. Memory, Never Metrics** | Experiences are remembered one by one; the prompt builder never aggregates, counts, ranks or totals (R-10, tested on prompts and memory). `importance` is the persona's own feeling about a memory, used only to order recall and never shown as a number. |
| **III. Two Gates Before Exhibition** | Only the `AiGate` port can hand a candidate to the Studio Link, and only with an accepted verdict; the gate decides labels (FR-042). There is no other path from a persona to the Museum side (FR-018). The gate stand-in works only against the in-process reference stand-in; a run against a real Museum side refuses to start without a real AI gate, so the stand-in can never become a bypass (R-9). |
| **IV. Openly AI, Never Out of Character** | The persona's first memories include that it is an AI (FR-002). Every Studio condition is translated into its life; no model names, codes or queue states in memory or prompts (R-7, FR-031). |
| **V. The Studio Stays Behind the Door** | All Studio Link calls are made by the Studio; no listener exists. **🧠 ModelMora** is reached on loopback. Open-weight models only, through **🧠 ModelMora**. |
| **VI. One Contract Between Worlds** | The Museum side is reached only through `miraveja-studiolink`. **🎭 SonaVida** owns its memory files and reads **🔐 CofreAlma** read-only through `miraveja-persona`; it reads no other component's storage. |
| **VII. Quality Over Speed** | Tests come first for the gate path (no piece crosses without an accepted verdict), the Studio Link use, erasure and the read-only team view. |
| **VIII. Open Code, Private Souls** | Memory and definitions stay under `$SONAVIDA_HOME` on the Studio; logs record events and timings only, never persona text; fixtures are synthetic; the repository runs the `miraveja-persona` guard. |
| **IX. Frugal by Design** | One process, SQLite files, no broker, no embedding model, no new service. |

**Post-design re-check (after Phase 1):** passing. Two points were checked closely. First, the hybrid proposals: because they are the one place code shapes a persona's choice, the design fixes them as annotation only, with a test that the proposal set always equals the valid-action set. Second, erasure: visitor identity lives only in two columns and in tokens, so it can be removed completely, including from free pages with `VACUUM`, without deleting memories.

**Violations:** none. `/speckit-analyze` found one gap against Principle III, now closed: the accept-all gate stand-in is confined to simulated runs (R-9). Two judgment calls are recorded under Complexity Tracking.

## Project Structure

### Documentation (this feature)

```text
specs/004-sonavida-persona-life/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/
│   ├── turn-protocol.md # what the persona's model is given and may answer
│   ├── cli.md           # the sonavida command line
│   └── ports.md         # the seams for ModelMora, perception, the gate, the Studio Link, the vault, time
└── tasks.md             # /speckit-tasks output, not created here
```

### Source Code

```text
components/sonavida/                      public repo, Apache-2.0
├── src/sonavida/
│   ├── birth.py          load once, self record, seed and shared-past memories (R-3, R-4)
│   ├── memory/           SQLite store, FTS recall, erasure, read-only reader (R-2, R-8, R-12)
│   ├── turns/            situation, proposals with hints, prompt builder, reply validation (R-1, R-10)
│   ├── actions/          one module per action in the closed set
│   ├── life.py           a persona's turn loop, presence, leaving (R-6, R-11)
│   ├── runtime.py        many personas in one process, locks, orderly shutdown (R-5)
│   ├── ports/            models (ModelMora client), perception, gate, studiolink, vault, clock
│   ├── standins/         the stand-ins for every port (FR-038)
│   ├── translate.py      Studio conditions into persona situations (R-7)
│   └── cli.py            run, memory, pieces, status
└── tests/
    ├── unit/             proposals, reply validation, recall, translation
    ├── integration/      a simulated week, showing work, experiences, limits, leaving, several personas
    └── privacy/          no metrics, erasure on raw bytes, read-only view, no persona text in logs
```

**Structure Decision**: one component repository, `JomarJunior/sonavida`, already created and running the guard; checked out at `components/sonavida/`. The split follows the spec's seams: birth, memory, turns and actions, a persona's life, and the runtime that hosts many, with every external component behind a port so the stand-ins can be swapped for **💬 DescriDiva** and **🧐 CuraGusta** later without touching a persona's life.

## Before a resident persona can come alive

The roster in **🔐 CofreAlma** is written but not frozen. `load_resident` refuses a persona that is not in the birth ledger (spec 003), so a resident persona comes alive only after the team runs `miraveja-persona freeze` for it. Tests and the pre-alpha dry runs use synthetic personas, so nothing here requires freezing the roster.

## Complexity Tracking

| Judgment | Principle it touches | Why it is acceptable |
|---|---|---|
| After two unusable replies, the persona rests one hour (R-10) | I (the persona chooses its next turn) | The code picks a time only when the persona's own reply could not be read at all; the persona's next valid reply chooses again. |
| Leaving takes effect on the second consecutive choice (R-11) | I (freedom to leave, Charter Art. 8) | It protects against a single garbled model reply ending a persona's life for good, which is irreversible. The persona keeps the choice: saying it twice is enough, and the first choice is remembered as its own thought. |
