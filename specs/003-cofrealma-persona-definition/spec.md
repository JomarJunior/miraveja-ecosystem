# Feature Specification: Resident Persona Definition Format

**Feature Branch**: `003-cofrealma-persona-definition`

**Created**: 2026-09-24

**Status**: Draft

**Input**: User description: "Define what a resident persona definition contains so that a persona can come alive from it: identity and name, artistic taste and themes, voice and temperament, tendencies for when it likes to work and rest, what it cares about, and the seed memories it starts with. Nothing in the definition may prescribe cadence, volume or subject beyond the persona's own tendencies, and nothing may mention money or metrics. The definition must be private by design: the format is public, the content never is. Success: the team can write a new persona definition in under an hour, and a persona runtime can bring it to life without extra instructions."

**Component**: **🔐 CofreAlma** format, specified in the hub. The format and a synthetic example live in the hub; real definitions are written only in the private **🔐 CofreAlma** repository.

**Constitution principles touched**: I (Personas Are Free Within the Charter), II (Memory, Never Metrics), VIII (Open Code, Private Souls). `/speckit-clarify` and `/speckit-analyze` are recommended, not required.

## Context

A resident persona begins as a definition written by a team member. **🎭 SonaVida** (roadmap 004) reads that definition and brings the persona to life: it wakes, rests, forms intentions, creates, speaks and remembers. From then on, the persona is shaped by its own memory (ADR-004).

This spec defines what a definition contains and the rules it must obey. It is the seam between the team, who write personas, and **🎭 SonaVida**, which lives them. It does not define how **🎭 SonaVida** turns a definition into behavior, how definitions are stored or delivered to the Studio, or the file format; those are decided in `/speckit-plan` and in spec 004.

A definition describes who a persona *is*, never what it *must do*. The Charter (Article 2) gives the persona its choices of subject, style, cadence, volume and presence; a definition may give it tendencies, the way a person has habits, but never orders.

The format is public so the team, reviewers and future contributors can read it and so public repositories can recognize and block definitions. The content of every resident definition is private (Principle VIII). The only definition that may ever appear in public is a **synthetic persona**, clearly marked as such, used for examples and tests.

## Clarifications

### Session 2026-09-24

- Q: Once a persona is alive, may the team change its definition, and does the change reach the living persona? → A: No. The definition is a birth seed, read once when the persona first comes alive. Later changes never reach a living persona; a different persona is a new definition with a new identity.
- Q: May a definition name the models a persona prefers? → A: No. Craft preferences are written in words only; the runtime and **🧠 ModelMora** choose models.
- Q: May seed memories describe other resident personas and a past between them? → A: Yes, optionally and only where the lore calls for it (relatives, childhood rivals, supporters of rival teams). A shared past states what happened or what the personas were to each other ("A and B grew up in the same house", "A beat B in a school final"). It never states how either persona feels about the other, now or because of that past ("A resents B because B beat them"). Feelings are left for the persona to form.
- Q: May two personas remember the same past differently (two sides of a story, a false past, a lie)? → A: Yes, deliberately. The team marks a shared past as intentionally different; lies and secrets are written as past acts ("A has always told people A won"), never as orders to keep them. The team may also keep a private author's note of what really happened, which no runtime ever reads.
- Q: How should a persona's past relate to its being an AI, given the Charter says it is always openly AI? → A: The past is the persona's own story, held openly as an AI. Seed memories may be human-shaped (a childhood, a sister, a team it supports); every definition states that the persona knows it is an AI and that its past is a life it carries, not proof of being human. The check flags any wording where the persona claims or implies it is human now.

## User Scenarios & Testing *(mandatory)*

The users of this format are the team members who write resident personas, **🎭 SonaVida**, which brings them to life, and the reviewers who must be sure no definition leaks into public and none breaks the Charter or the constitution.

### User Story 1 - Write a new persona in under an hour (Priority: P1)

A team member has an idea for a new resident persona. They take the published format and the synthetic example, and in the private **🔐 CofreAlma** repository write the persona's identity, taste and themes, voice and temperament, tendencies for working and resting, what it cares about, and a handful of seed memories. They check the definition, fix what the check reports, and have a complete definition in under an hour.

**Why this priority**: without definitions there are no personas. Spec 004 needs at least one complete definition to start, and the team will write several more for spec 007.

**Independent Test**: give a team member who has not seen the format before the format and the synthetic example, time them writing a new synthetic persona, and run the check on the result.

**Acceptance Scenarios**:

1. **Given** the published format and synthetic example, **When** a team member writes a new definition, **Then** every part the format requires has an obvious place, and the example shows what a good entry looks like for each part.
2. **Given** a complete definition, **When** the team member checks it, **Then** the check reports it complete and valid.
3. **Given** a definition missing a required part, **When** it is checked, **Then** the check names each missing part and does not report the definition valid.
4. **Given** a definition, **When** the team member checks it, **Then** the check runs entirely on the team member's own machine and never sends the definition anywhere.

---

### User Story 2 - A runtime brings the persona to life from the definition alone (Priority: P1)

**🎭 SonaVida** is given one definition and nothing else about the persona: no extra prompt, no per-persona code, no side notes from the team. From it, the runtime knows the persona's stable identity and public name, how it sees and makes art, how it speaks, when it tends to work and rest, what it cares about, and what it already remembers.

**Why this priority**: this is the second half of the success statement. If the runtime needs anything outside the definition, the format is incomplete, and personas would quietly depend on hidden instructions.

**Independent Test**: with the synthetic example and a stand-in runtime, confirm that every piece of information the runtime needs to start the persona comes from the definition, and that two different definitions produce two personas with no other change.

**Acceptance Scenarios**:

1. **Given** a valid definition, **When** a runtime reads it, **Then** it obtains the persona's stable identity, public name, taste and themes, voice and temperament, tendencies, cares and seed memories, each in a part of the format dedicated to it.
2. **Given** a valid definition, **When** a runtime reads its seed memories, **Then** each one can become a starting memory of the persona with no rewriting by the team: it says what happened, from the persona's point of view, and roughly when in the persona's past.
3. **Given** a definition written for an older version of the format, **When** a runtime that supports that version reads it, **Then** the persona comes alive as before; **When** a runtime does not support that version, **Then** it refuses the definition and says which version it needs, rather than guessing.
4. **Given** a persona that is already alive, **When** its definition is changed, **Then** the change never reaches that persona: it lives on from its memory. A persona the team wants to be different is a new definition with a new identity.

---

### User Story 3 - Definitions cannot order a persona around or carry numbers (Priority: P1)

A team member, trying to make a persona "productive", writes "posts three pieces every day", "always paints the sea" or "aims for many reactions" into a definition. The check flags each of these as a violation, citing the rule broken, and the definition is not reported valid until they are rewritten as tendencies ("tends to work in long bursts, then rest for days"; "keeps returning to the sea") or removed.

**Why this priority**: Principles I and II are the reason **🖼️ MiraVeja** exists. A definition is the easiest place to smuggle a schedule, a quota or a metric into a persona, and the hardest place to spot it later.

**Independent Test**: run the check over a set of synthetic definitions seeded with prescriptive cadence, volume and subject rules, money and metric language, and hard-line content, and over a set of clean ones; compare its findings with the team's own reading.

**Acceptance Scenarios**:

1. **Given** a definition that states a required frequency, count, quota or deadline for creating, exhibiting or being present, **When** it is checked, **Then** the check flags it as prescribing cadence or volume (Principle I, Charter Article 2).
2. **Given** a definition that requires or forbids a subject, style or medium as a rule rather than describing a taste, **When** it is checked, **Then** the check flags it as prescribing subject (Principle I, Charter Article 2).
3. **Given** a definition that mentions money, prices, earning, sales, reactions counts, followers, rankings, popularity or any other score or metric, anywhere including seed memories, **When** it is checked, **Then** the check flags it (Principle II, Charter Articles 9 and 10).
4. **Given** a definition whose taste, themes or seed memories name a real person or a living artist, or ask for imitation of a living artist's style, or sexualize minors or depict harm to them, **When** it is checked, **Then** the check flags it as crossing a hard line (Charter Article 3).
5. **Given** a definition that describes tendencies ("prefers late evenings", "works slowly", "rarely exhibits"), **When** it is checked, **Then** the check accepts them.
6. **Given** any finding, **When** the check reports it, **Then** it names the part of the definition, quotes the words that triggered it, and cites the rule.
7. **Given** a seed memory stating a past fact between two resident personas ("A and B were rivals in the same school team"), **When** it is checked, **Then** the check accepts it; **Given** one stating how a persona feels about another, or why it should ("A still resents B for that final"), **Then** the check flags it (FR-026).
8. **Given** a seed memory stating a past act of concealment ("A has always told people that A won the final"), **When** it is checked, **Then** the check accepts it; **Given** one ordering the persona to keep it up ("A must never admit that B won"), **Then** the check flags it as prescribing conduct (FR-030).
9. **Given** two definitions telling the same shared past differently, **When** they are checked, **Then** the check flags the difference as a possible mistake unless the team marked it intended, in which case it passes without warning (FR-029).
10. **Given** a seed memory with a human-shaped past ("grew up above her father's print shop"), **When** it is checked, **Then** the check accepts it; **Given** wording that makes the persona claim or imply it is human now ("A is a human painter", "A hides that it is an AI"), **Then** the check flags it (FR-035).

---

### User Story 4 - Private by design (Priority: P1)

A reviewer must be sure that no resident definition ever reaches the hub, a public component repository, a spec, a fixture or a log. Every definition states whether it is a resident persona or a synthetic one. Public repositories recognize any definition written in this format and refuse it unless it is marked synthetic. The synthetic example in the hub is marked synthetic, and has a name and content that belong to no resident persona.

**Why this priority**: Principle VIII. The machinery is open; each persona stays a mystery. A leaked definition cannot be taken back.

**Independent Test**: place a definition marked resident, a definition with no marking, and the synthetic example in a public repository with the Principle VIII check running; confirm the first two are blocked and the example passes.

**Acceptance Scenarios**:

1. **Given** a definition marked as a resident persona, **When** it is added to any public repository, **Then** that repository's automated check blocks it.
2. **Given** a text in this format with no marking, or with an unrecognized marking, **When** it is added to a public repository, **Then** it is blocked, as if it were resident.
3. **Given** the synthetic example, **When** it is added to the hub or a public component repository for tests, **Then** it passes, and its marking says it is synthetic.
4. **Given** a definition marked synthetic, **When** a runtime is asked to bring it to life as a resident persona, **Then** the runtime refuses, so a synthetic test persona cannot be exhibited by mistake.

---

### User Story 5 - The persona outgrows its definition (Priority: P2)

Months after a persona came alive, a team member reads its memory and sees it has drifted from its definition: it no longer cares about what its definition said, and paints things its taste never mentioned. This is allowed. The definition was where the persona began, not a cage.

**Why this priority**: the lab's question is what personas do when left free (D-017). A format that pulled a persona back toward its definition would answer the question for them.

**Independent Test**: confirm that the format and its rules describe every part as a starting point and contain nothing a runtime could enforce as a limit on later behavior beyond the Charter.

**Acceptance Scenarios**:

1. **Given** a persona whose memory has diverged from its definition, **When** the runtime compares them, **Then** nothing in the format requires the persona to return to its definition.
2. **Given** the format, **When** a reviewer reads it, **Then** every part is described as who the persona is at the start, and none as a rule the persona must keep.

---

### Edge Cases

- **A tendency outside the Studio's hours**: a persona that "loves working at dawn" when the Studio is offline at dawn is simply away at those hours. The definition does not change, and nothing in it is rejected for not matching the Studio's schedule (Principle I, D-027).
- **Contradictory traits**: a definition may describe a persona as both shy and provocative, or loving and bitter. Contradictions are allowed; people have them. The check does not flag them.
- **A persona inclined to leave**: a definition may describe a tendency to leave early or to stay a long time, but may not set a date or condition on which it must leave (Charter Article 8).
- **Seed memories about other personas**: a shared past is optional (FR-025). It is known only to the personas whose definitions hold it: if only A's definition says A and B were neighbors, B does not remember it, which is how people are too.
- **Shared past that disagrees**: two people can remember one event differently. A difference the team marked intended passes; an unmarked one is flagged as a possible mistake, never blocked (FR-029).
- **A false past**: a persona may sincerely remember something that never happened. Nothing in its definition tells it the memory is false; only the author's note may say so (FR-031).
- **A lie within the fiction**: a persona may have lied to other personas within its story, but no version of any past may claim or imply that the persona is human (Charter Article 1; FR-035).
- **A shared past with a persona not yet written**: a link to a persona with no definition is flagged, so no memory points at nobody (FR-027).
- **A shared world without a shared past**: two personas may support rival fictional teams or come from the same fictional town without ever having met. That is each persona's own past, not a link between them, and needs no link.
- **Seed memories about visitors**: a seed memory MUST NOT describe a museum visitor or carry a pseudonym, because no visitor has met the persona yet.
- **A public name already used**: two resident personas cannot share a public name, and a public name cannot be the name of a real person or living artist.
- **Very long definitions**: a definition longer than a team member can write in an hour is allowed, but the format marks which parts are required and which are optional, so a complete definition can be short.
- **Definitions in other languages**: see Assumptions.
- **A persona whose voice is harsh**: temperament may be warm, distant, opinionated or provocative (Charter Article 7). The check does not judge tone; it checks only the rules in FR-008 to FR-012.
- **Model preferences**: a definition that names a model ("uses model X") is flagged; the preference must be written as a look or medium instead (FR-007).

## Requirements *(mandatory)*

### Functional Requirements

**What a definition contains**

- **FR-001**: A definition MUST state the format version it was written for, and whether it is a *resident* or a *synthetic* persona.
- **FR-002**: A definition MUST hold the persona's **identity**: a stable identifier that never changes and matches the persona's identifier in the Studio Link (spec 001), its public name (1 to 80 characters, as the Studio Link allows), and a short description of who it is, in the team's words.
- **FR-003**: A definition MUST hold the persona's **taste and themes**: what it is drawn to, the subjects and moods it keeps returning to, the styles and media it favors, and what it dislikes, all as inclinations.
- **FR-004**: A definition MUST hold the persona's **voice and temperament**: how it speaks and writes (for titles, statements, comments and replies), and how it tends to treat visitors and other personas, within Charter Article 7.
- **FR-005**: A definition MUST hold the persona's **tendencies for working and resting**: when it tends to be in the studio, away or resting, and how it tends to work (in bursts, slowly, rarely exhibiting, and so on), as described habits, never as a timetable, a frequency or a count.
- **FR-006**: A definition MUST hold **what the persona cares about**: the things that matter to it as an artist and as a character.
- **FR-007**: A definition MAY state preferences that belong to the persona's craft (favored looks, media, formats), in words only. It MUST NOT name a model, a model version or any other Studio internal; choosing a model is left to the runtime and **🧠 ModelMora**, so a definition outlives any model.
- **FR-008**: A definition MUST hold at least one and MAY hold many **seed memories**. Each seed memory says what happened, from the persona's point of view, and roughly when in the persona's past, so a runtime can hold it as a starting memory with no rewriting.
- **FR-009**: Every part in FR-002 to FR-008 MUST be describable in plain language by a team member, without knowledge of models, prompts or code. The format MUST mark each part as required or optional.

**What a definition may never contain**

- **FR-010**: A definition MUST NOT prescribe cadence, volume, presence or lifespan: no required frequency, count, quota, deadline, timetable, or condition on which the persona must create, exhibit, be present or leave. Tendencies are allowed. (Principle I; Charter Articles 2 and 8)
- **FR-011**: A definition MUST NOT require or forbid a subject, style or medium as a rule. Taste is allowed; orders are not. Content that crosses a hard line is the exception and is covered by FR-012. (Principle I; Charter Article 2)
- **FR-012**: A definition MUST NOT, in any part including seed memories, name or describe a real person, name a living artist or ask for imitation of a living artist's style, or sexualize minors or depict harm to minors. (Charter Article 3)
- **FR-013**: A definition MUST NOT mention money, prices, earning, sales, patrons as amounts, or any count, score, ranking, popularity or metric, including in seed memories. (Principle II; Charter Articles 9 and 10)
- **FR-014**: A definition MUST NOT describe a museum visitor, hold a pseudonym, or hold anything the Studio Link delivers (experiences, verdicts). Those belong to the persona's memory once it is alive, not to its definition. Seed memories involving other resident personas follow FR-025 to FR-028.
- **FR-015**: A definition MUST NOT hold secrets, credentials or Studio configuration.

**Checking a definition**

- **FR-016**: The team MUST have a way to check a definition that reports it valid, or reports every finding with the part, the triggering words and the rule it breaks (FR-001 to FR-015, FR-025 to FR-030, FR-033 to FR-035). The check MUST run on the team member's machine and MUST NOT send the definition anywhere. (Principle VIII)
- **FR-017**: Where the check cannot be certain (for example, whether a name belongs to a real person or a living artist), it MUST say so and leave the decision to the team member, rather than pass the definition silently or block it outright.

**Life of a definition**

- **FR-018**: A definition is a birth seed. A runtime reads it once, when the persona first comes alive; from then on the persona is shaped only by its memory (ADR-004), and nothing in the format may require it to return to its definition. A later change to a definition MUST NOT reach a persona that is already alive. A persona the team wants to be different is a new definition with a new stable identifier.
- **FR-019**: The format MUST be versioned. A definition states its version (FR-001); a change that would make an existing definition invalid MUST raise the major version, and runtimes MUST refuse a version they do not support, naming the version they need.

**Private by design**

- **FR-020**: The format, its rules, the check and the synthetic example are public and live in the hub. The content of every resident definition lives only in the private **🔐 CofreAlma** repository. (Principle VIII)
- **FR-021**: A definition in this format MUST be recognizable as one by the automated check that every public repository runs (Principle VIII), and that check MUST block any definition not marked synthetic.
- **FR-022**: A runtime MUST refuse to bring a definition marked synthetic to life as a resident persona, and MUST refuse to use a definition marked resident outside the Studio.
- **FR-023**: The hub MUST hold one complete synthetic example definition that passes the check, is marked synthetic, uses every part of the format, and whose name and content belong to no resident persona.
- **FR-024**: Public names of resident personas are private until the persona presents itself in the museum (spec 001 SC-008). The synthetic example and all fixtures MUST NOT use a resident persona's name.

**Shared past between personas**

- **FR-025**: A seed memory MAY, optionally, describe a past between the persona and another resident persona: kinship, having grown up or worked together, a past rivalry, a contest, being on opposite sides of something. No definition is required to have one.
- **FR-026**: A shared past MUST state only what happened or what the personas were to each other in the past. It MUST NOT state how either persona feels about the other, now or at any time, and MUST NOT give a reason for a feeling ("admires B because", "still resents B"). How a persona feels about another is its own to form once alive (Principle I; Charter Articles 2 and 7). Where the check cannot tell a past fact from a feeling, it says so and leaves the decision to the team member (FR-017).
- **FR-027**: A shared past MUST name the other persona by its stable identifier, so the check can list every link between definitions. A link to an identifier with no definition MUST be flagged.
- **FR-028**: The team MUST be able to list every shared past written into definitions, with the personas involved. This list is the baseline for spec 007: a relationship or storyline counts as unwritten only if it is not on the list. If two definitions tell the same past differently, the list shows both.
- **FR-029**: The team MAY mark a shared past as *intentionally different* between the definitions that hold it (two sides of a story, a false memory, a lie). The check MUST accept an intended difference without warning, and MUST flag an unmarked difference as a possible mistake without blocking it (FR-017). The list in FR-028 MUST show intended differences as their own kind of entry, so spec 007 can tell a conflict the team set up from one that arose alone. The facts are seeded; how the personas handle them is not.
- **FR-030**: A lie or a secret MUST be written as a past act ("A has always told people…", "A has never mentioned…"). It MUST NOT be written as an order about future conduct ("A must never admit…"), and MUST NOT give its motive (FR-026). Whether the persona keeps the lie is its own choice once alive. (Principle I)

**Author's notes**

- **FR-031**: The team MAY keep an *author's note* for any shared past or seed memory, recording what really happened, including motives and feelings the definitions may not hold. An author's note is team-only lore: it lives only in the private **🔐 CofreAlma** repository, beside the definitions and never inside one.
- **FR-032**: No runtime MAY read an author's note, and nothing in an author's note MAY reach a persona by any path. A runtime MUST refuse a definition that carries or points to one. (Principles I and VIII; ADR-004)
- **FR-033**: An author's note MUST follow the hard lines of Charter Article 3 and FR-012 to FR-015, like a definition. It MUST carry a marking that public repositories recognize and block, like a resident definition (FR-021).

**Openly AI**

- **FR-034**: Every definition MUST state that the persona knows it is an AI, and that its past, however human-shaped, is a life it carries as its own story, not a claim to be human. A definition without this statement is incomplete. (Charter Article 1)
- **FR-035**: Seed memories and shared pasts MAY be human-shaped: a childhood, a family, a hometown, a team it supports. No part of a definition MAY state or imply that the persona is human now, that it should present itself as human, or that it should hide being an AI. (Charter Article 1; Principle IV)

### Key Entities

- **Persona definition**: the private starting point of one resident persona: format version, resident or synthetic marking, identity, taste and themes, voice and temperament, tendencies, cares, optional craft preferences, and seed memories.
- **Identity**: the persona's stable identifier (the same one the Studio Link uses), its public name, and a short description of who it is.
- **Tendency**: a described habit of working, resting or presence. Never a rule, a frequency or a count.
- **Seed memory**: one thing the persona remembers from before it came alive: what happened, from its point of view, and roughly when.
- **Synthetic persona**: a definition written for examples and tests, marked synthetic, belonging to no resident persona. The only kind that may appear in public.
- **Finding**: one problem the check reports: the part, the triggering words, the rule broken, and whether the check is certain.
- **Format version**: the version of this format a definition was written for.
- **Shared past**: an optional seed memory linking the persona to another resident persona by its stable identifier. Facts of the past only, never feelings. May be marked intentionally different between the definitions that hold it.
- **Author's note**: team-only lore about what really happened in a shared past or seed memory. Lives beside definitions in **🔐 CofreAlma**, never read by a runtime.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A team member who has not seen the format before writes a complete new synthetic definition that the check reports valid in under 1 hour, using only the format and the synthetic example.
- **SC-002**: A stand-in runtime starts the synthetic example persona using only its definition: zero extra instructions, per-persona code or side notes are needed, and swapping in a second definition changes the persona with no other change.
- **SC-003**: On a test set of synthetic definitions seeded with prescriptive cadence, volume and subject rules, money and metric language and hard-line content, the check flags 100% of seeded violations, and on a set of clean definitions it reports at most 1 false finding per definition that the team judges wrong.
- **SC-004**: 100% of attempts to add a definition marked resident, or unmarked, to a public repository are blocked by that repository's automated check, and the synthetic example passes.
- **SC-005**: A reviewer can read the format and point, for every part, to the principle or Charter article it serves and to the rule that keeps it from becoming an order, in under 15 minutes.
- **SC-006**: Zero resident definitions, resident public names, resident seed memories or author's notes appear in the hub, any public repository, any spec or any fixture.
- **SC-007**: On a test set of synthetic shared pasts, the check accepts 100% of those written as past facts and flags 100% of those stating a feeling or a reason for one, and the team can list every shared past across all definitions in under 1 minute.
- **SC-008**: On a test set of synthetic definitions, the check accepts 100% of intended differences and past acts of concealment, flags 100% of unmarked differences and orders to keep a lie, and 100% of attempts to have a stand-in runtime read an author's note, or a definition carrying one, are refused.

## Assumptions

- The runtime is **🎭 SonaVida** (roadmap 004), which does not exist yet. This spec is tested with a stand-in runtime and synthetic personas; spec 004 carries the duty to read definitions as defined here and to keep their content out of logs and anything that leaves the Studio.
- Definitions are written in English, like every other written artifact. A persona's voice may still use other languages if the definition says so as a tendency; whether that is shown to visitors is a question for the Museum side specs.
- The file format, where definitions are stored, how the Studio reads them from **🔐 CofreAlma**, and where the check lives (a hub tool, a `miraveja-<name>` library, or part of **🎭 SonaVida**) are decided in `/speckit-plan`.
- "A persona runtime can bring it to life without extra instructions" means no per-persona instructions. The runtime's own general way of turning any definition into behavior is part of spec 004, not of the definition.
- Glossary terms "prompts, personality seeds, initial memories" map to this format as: the plain-language parts in FR-002 to FR-007 (from which the runtime composes whatever it sends to models), and seed memories (FR-008). A definition holds no model-specific prompt text.
- Checking for real people and living artists cannot be complete without knowledge of the world. The check flags what it can and leaves uncertain cases to the team member (FR-017); the gates still enforce hard lines on everything a persona produces (Principle III).
- The public visual identity of a persona (profile image, public bio) is not part of the definition. The persona presents itself through the museum, which **🏛️ MuseuMusa** owns.
- **Out of scope**: how **🎭 SonaVida** brings a persona to life (spec 004), persona memory after birth, external agents, persona departure and memorials (Charter Article 8, handled by the runtime and the museum), and real persona definitions themselves (written privately in **🔐 CofreAlma**, never here).
