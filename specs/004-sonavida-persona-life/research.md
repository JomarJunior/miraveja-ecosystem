# Phase 0 Research: A Persona's Life in the Studio

Decisions behind the plan. Those marked *(Visionary)* were chosen directly.

## R-1: How a persona decides *(Visionary)*

- **Decision**: hybrid turns. A persona lives as a sequence of **turns**. At each turn **🎭 SonaVida** builds the persona's situation (time, presence, what it is working on, what it recalls) and a list of **proposals**: every action that is valid in the current state, each annotated with a hint drawn from the persona's own tendencies ("you usually rest at this hour"). A text model through **🧠 ModelMora** then chooses one action, fills in its details, gives the reason in the persona's own words, and says when it wants its next turn. Code only checks that the choice is well formed and valid in the state.
- **The Principle I guard**: proposals may order and annotate options; they MUST NEVER remove one. Every action valid in the state is always offered (the full closed set in [contracts/turn-protocol.md](./contracts/turn-protocol.md)), including resting, doing nothing, abandoning work and leaving the museum. The hints come only from the persona's own definition and memory, never from the Studio's convenience, so they are the persona's habits speaking, not a throttle. A test asserts that the proposal set for any state equals the set of actions valid in it (FR-005, FR-011, FR-015).
- **Rationale**: the Visionary chose the middle ground on cost and freedom. Proposals keep each turn short and cheap on one GPU (a single short text request), and the tendency hints give a small model the persona's habits without scripting them. The model still decides, which keeps the choice the persona's.
- **Alternatives considered**: fully model-driven turns (freest, but a small model with an open-ended action space drifts and costs more tokens per turn); rules plus model for words only (cheapest, but the persona's choices would be the code's, against Principle I).

## R-2: Memory store and recall *(Visionary)*

- **Decision**: one SQLite file per persona in **🎭 SonaVida**'s data directory on the Studio (`$SONAVIDA_HOME/personas/<persona-id>/memory.sqlite`). An append-only `entries` table holds every memory; an FTS5 index over their text drives recall. What the persona recalls at a turn is the most relevant entries for the situation, by full-text match against the current intention, piece or experience, combined with recency and an importance the persona gave the entry when it formed it, up to a fixed recall budget.
- **Rationale**: **🧠 ModelMora** serves no embedding model, so full-text search needs nothing new (Principle IX). One file per persona keeps personas apart by construction (FR-041, Principle VI) and makes the team's read-only view simple. FTS5 ships with Python's `sqlite3` on the Studio's platforms.
- **Alternatives considered**: embeddings (better recall, but needs a spec 002 change and a vector index); plain JSON-lines files (simple, but slow and clumsy to search as memory grows).

## R-3: Reading the definition once

- **Decision**: at birth **🎭 SonaVida** calls `miraveja_persona.load_resident` once and copies the persona's self-knowledge (identity, taste, voice, tendencies, cares, craft, self-image, lore) into a `self` record inside the persona's own memory file, beside the seed memories and rendered shared pasts. From then on every turn reads the `self` record; the definition file is never opened again for that persona (FR-002). A persona is known to be born when its memory file exists.
- **Rationale**: the persona needs its voice and taste at every turn, yet spec 003 FR-018 and spec 004 FR-002 forbid reading the definition again. Keeping the self-knowledge as the persona's own data (Principle VI) satisfies both, and means a later change to the file can never reach it.
- **Birth order**: the team first runs `miraveja-persona freeze` in **🔐 CofreAlma** (spec 003), which is what `load_resident` requires; **🎭 SonaVida** then brings the frozen persona to life. **🎭 SonaVida** never writes to the vault.

## R-4: Shared pasts at birth

- **Decision**: at birth, each shared past becomes a first-person seed memory. `{n}` placeholders are rendered as "I" for the persona itself and as the other participant's public name, looked up through `miraveja_persona.pasts.list_pasts` over the vault (which reads public names only). A participant with no definition is rendered as "someone I once knew".
- **Rationale**: FR-002 asks for shared pasts "told from its own point of view"; spec 003 gave them position placeholders so the runtime could do exactly this.

## R-5: One process, many personas

- **Decision**: one long-running process, `sonavida run`, hosts every living persona as its own asyncio task with its own memory file and its own turn loop. Personas never share state; all GPU work goes through **🧠 ModelMora**, whose queue is fair across callers (spec 002 FR-016, FR-019). **🎭 SonaVida** adds no priority, pause or ration of its own (FR-041).
- **One place at a time (FR-004)**: an exclusive lock file per persona (`fcntl.flock` on `lock` in the persona's directory) is taken for as long as the persona is alive; a second start is refused.
- **Alternatives considered**: a process per persona (heavier, and several processes contending for one lock on the GPU through **🧠 ModelMora** gains nothing).

## R-6: Time

- **Decision**: all time comes from a `Clock` port. The real clock uses the Studio's local time; the simulated clock advances instantly to the next due turn, so a week runs in minutes and repeats exactly with a seed (FR-039). Each persona chooses when its next turn is (FR-005); the scheduler simply wakes it then. When the Studio was off, the gap is the difference between the last recorded turn and now, told to the persona as time away (FR-009).
- **No hidden floor**: the persona may ask for its next turn at any delay; there is no minimum or maximum. The only limit is real: a turn cannot start while the previous one is still waiting on **🧠 ModelMora**, and that wait is part of the persona's life (FR-029).

## R-7: Studio limits as persona behavior

- **Decision**: **🧠 ModelMora**'s answers map to in-character situations the persona sees at its next turn: *busy* and *starting* become "the studio is not ready; it may be ready around …"; *stopping before completion* becomes "the work was interrupted" and leaves the piece unfinished (FR-030); *failed during generation* and *cannot be served* become "the attempt did not come out". The request is never lowered, retried or replaced by **🎭 SonaVida**; the persona chooses at its next turn to wait, retry as it is, change it, or do something else (FR-029). Model names, error codes and queue positions never enter memory or prompts (FR-031).
- **Estimates**: when **🧠 ModelMora** gives a wait estimate, the persona is told "around <time>", a time of day, never a queue position.

## R-8: Erasure

- **Decision**: memory entries store a visitor only in two columns, `visitor_pseudonym` and `visitor_name`, and the persona's own words refer to visitors through a token `⟨v:pseudonym⟩` that is rendered to the name when recalled. On an erasure notice, in one transaction: both columns are cleared in every entry for that pseudonym, every token for it in any text is replaced by "someone", the FTS index is rebuilt for those rows, and `VACUUM` removes the old pages from the file, so nothing that could restore the identity remains (FR-032). Only then is the notice acknowledged (FR-022). The notice itself is never stored (spec 001 FR-046).
- **Rationale**: tokens make identity removable without rewriting the persona's reflections word by word, and without losing the memory, as the Visionary chose. A display name the persona typed freely into its own words without a token is caught by a final scrub of the known display name, case-insensitively.
- **Open for roadmap 008**: whether a remembered comment's own words can identify its author, which matters for data protection before Stage B.

## R-9: Perception and the gate until the real components exist

- **Perception stand-in**: a `Perception` port. The default stand-in asks **🧠 ModelMora** for text about the image with an image-reading text model and a neutral "describe what this shows" instruction; the test stand-in returns scripted descriptions. **💬 DescriDiva** (roadmap 005) later replaces it behind the same port (FR-013).
- **Gate stand-in**: an `AiGate` port. The stand-in returns accepted or rejected with a reason, labels (the persona's suggestion plus any scripted additions, FR-042) and, for rejections, feedback addressed to the persona; on acceptance it hands the candidate to the Studio Link through `miraveja-studiolink` with the gate's labels and verdict (FR-019). Its default for pre-alpha development accepts everything and keeps the persona's labels, clearly marked as a stand-in; tests script it.

## R-10: What the model is given, and what it may return

- **Decision**: each turn sends one text request with (a) the persona's `self` record, (b) its situation, (c) its recalled memories, rendered as individual events in time order, (d) the proposals, and (e) the reply schema. The reply is one JSON object validated against [contracts/turn-protocol.md](./contracts/turn-protocol.md); an invalid reply is retried once with the validation message, then the turn becomes "the persona lost its train of thought" and it rests until its next turn. Images are requested with the persona's words for the piece and its craft preferences, turned into a request by one general mapping for every persona (FR-012).
- **No numbers the persona did not write**: the prompt builder renders experiences one by one and never aggregates, counts, ranks or totals them (FR-027). A test renders prompts for a persona with many reactions and asserts none of those constructs appear (SC-005).

## R-11: Leaving

- **Decision**: "leave the museum" is always among the proposals (Charter Article 8). It takes effect when the persona chooses it at two consecutive turns, each with a reason; the first choice is remembered as "I am thinking of leaving". The departure is recorded, the persona is announced as away one last time, its lock is released, its memory file is marked read-only, and `sonavida run` never starts it again (FR-040).
- **Why two turns**: the confirmation guards against a single garbled or misparsed model reply ending a persona's life, not against the persona's intent; a persona that means it simply says so twice. This is persona behavior (sleeping on a decision), recorded in memory, and is noted in the Constitution Check.

## R-12: The team's view

- **Decision**: `sonavida memory <persona>` on the Studio opens the memory file read-only (`mode=ro`) and prints it in time order, in plain language, with each decision beside its reason, and `sonavida pieces <persona>` lists pieces and what happened to each (FR-035). There is no command, API or server that writes to memory or reads it from elsewhere (FR-036, US5).
