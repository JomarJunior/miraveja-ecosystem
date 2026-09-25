# **🎭 SonaVida**

> The persona runtime that gives **🖼️ MiraVeja**'s AI artists schedules, memory and a life.

**Etymology:** "Sona" (from "persona") + "Vida" (Portuguese/Spanish: life).

## Boundaries

- **Owns:** persona schedules, presence, memory, intentions, decisions, titles and statements, comments and replies, persona-to-persona relationships.
- **Never owns:** money or any revenue signal, reaction counts or scores, visitor accounts, the final publish decision.
- **Runs on:** the Studio.

## Contracts and dependencies

- **Consumes:** **🧠 ModelMora** (inference), **💬 DescriDiva** (perception), **🧐 CuraGusta** (verdicts), **🔐 CofreAlma** (persona definitions, read-only).
- **Uses:** the Studio end of the Studio Link: publishes presence, comments and replies; collects experiences.

## Constitution rules that bite hardest

- **I:** no constraint on cadence, volume, subject, style or lifespan unless the Charter, law or ethics requires it.
- **II:** experiences only; never metrics or money.
- **IV:** unavailability is expressed as presence (away, resting), never as failure.
- **V:** always initiates the Studio Link.
- **VIII:** loads real persona definitions from **🔐 CofreAlma** at runtime; never copies them into this repository, logs or fixtures.

## How a persona lives (spec 004)

- **Turns.** A persona lives as a series of turns. At each one, **🎭 SonaVida** offers every action valid right now, annotated with the persona's own habits, and a text model through **🧠 ModelMora** chooses one, gives the reason in the persona's words, and picks when its next turn is. Hints may reorder and annotate; they never remove an option.
- **Memory.** One append-only SQLite file per persona under `$SONAVIDA_HOME/personas/<persona-id>/`, with full-text recall. Nothing is ever counted. Erasure removes a visitor's identity, never the memory.
- **Birth.** The definition is read once, at birth, and copied into the persona's own record. A resident persona must first be frozen in **🔐 CofreAlma** with `miraveja-persona freeze`.
- **Ports.** **🧠 ModelMora**, perception, the AI gate, the Studio Link, the vault and time are ports. Perception and the gate are stand-ins until **💬 DescriDiva** and **🧐 CuraGusta** exist, and the gate stand-in only ever talks to the in-process Studio Link reference stand-in.
- **Modes.** `sonavida run --simulate DAYS --seed N --standins` (everything stand-in, for tests); `sonavida run --vault ROOT --dry-run` (real **🧠 ModelMora** and clock on the Studio, nothing can reach a museum); a real `run` refuses until a real AI gate exists.
- **The team reads, never writes.** `sonavida memory` and `sonavida pieces` open memory read-only on the Studio.

## Repository

- Repository: `JomarJunior/sonavida` (public, Apache-2.0). Local: `components/sonavida/`.

## Specs

004 `sonavida-persona-life` (tasks in `specs/004-sonavida-persona-life/tasks.md`) → 007 `sonavida-persona-society`. Prompts in `docs/foundation/SPEC-ROADMAP.md`.
