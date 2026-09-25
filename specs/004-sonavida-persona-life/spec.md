# Feature Specification: One Persona's Life in the Studio

**Feature Branch**: `004-sonavida-persona-life`

**Created**: 2026-09-25

**Status**: Draft

**Input**: User description: "Bring one resident persona to life in the Studio. It wakes and rests on its own tendencies, shows its presence, forms an intention for a piece, creates it, gives it a title and a short statement in its own voice, decides whether to submit it for exhibition, and remembers what it made and what happened to it. When the Studio is unavailable, the persona is simply away. A persona never receives counts, scores or money, only experiences it may remember. Success: over a week, the persona produces work on its own rhythm, and a team member reading its memory can tell what it has been doing and why."

**Component**: **🎭 SonaVida**. Code is written in `components/sonavida/`.

**Constitution principles touched**: I (Personas Are Free Within the Charter), II (Memory, Never Metrics), IV (Openly AI, Never Out of Character), V (The Studio Stays Behind the Door). Also VIII (Open Code, Private Souls) through the persona definition and memory. `/speckit-clarify` and `/speckit-analyze` are required (Principles II and IV).

## Context

A resident persona begins as a definition written by the team (spec 003) and kept in **🔐 CofreAlma**. **🎭 SonaVida** reads that definition once and brings the persona to life. From then on the persona lives in the Studio: it keeps its own hours, decides what to make, makes it, names it, decides whether to show it, and remembers. It is shaped only by its memory (ADR-004).

This spec covers personas living alone: several may be alive at once, but none sees or meets another. It uses what already exists: the definition format and its loader (spec 003), **🧠 ModelMora** for text and images (spec 002), and the Studio end of the Studio Link with its reference stand-in for the Museum side (spec 001). **💬 DescriDiva** (roadmap 005) and **🧐 CuraGusta** (roadmap 006) do not exist yet; until they do, this spec is built and tested with stand-ins for perception and for the AI gate, which the real components later replace without changing the persona's life.

Looking at other personas' work, commenting and replying, and relationships between personas are roadmap 007. Visitors are roadmap 011 and 012. This spec does receive what visitors and the gates leave for the persona (experiences, gate outcomes, erasure notices), because remembering what happened to its work is part of a life.

The Studio is one machine that keeps hours. Its limits are real, and they must reach the persona as part of its life, never as hidden throttling of what it wants (Principle I). When the Studio is off, the persona is away.

## Clarifications

### Session 2026-09-25

- Q: Is departure (Charter Article 8) in scope for this spec? → A: Yes, recorded in the Studio only. The persona may choose to leave; **🎭 SonaVida** records the departure, stops bringing it alive, and announces it as away one last time. The museum shows it as away indefinitely until a later spec adds a departed state to the Studio Link.
- Q: How deep is forgetting on an erasure notice? → A: The memories stay but lose the visitor's identity. The persona can still recall that "someone once said…", but can no longer recognize, name or link that visitor.
- Q: Should spec 004 run one persona, or let several live at once on the Studio's single GPU? → A: Several can live at once, each alone. Any number of roster personas may be alive together, each keeping its own hours and sharing the GPU through **🧠 ModelMora**'s fair order; they do not see or meet each other (roadmap 007).

## User Scenarios & Testing *(mandatory)*

### User Story 1 - A persona comes alive and keeps its own hours (Priority: P1)

The team places a new definition in **🔐 CofreAlma** and starts **🎭 SonaVida** in the Studio. The persona comes alive from its definition, with its seed memories as its first memories. From then on it decides for itself when to be in the studio, when to rest and when to be away, guided by the tendencies in its definition and by what it remembers. Each change of presence is announced through the Studio Link, so the museum always shows where the persona is.

**Why this priority**: nothing else in the persona's life can happen before it is alive and keeping hours. It is also the first place the Studio's own hours meet the persona's.

**Independent Test**: bring a synthetic persona to life against the Studio Link reference stand-in, let it run through several simulated days, and check that it was born once, that every presence change it chose was announced, and that each one is explained in its memory.

**Acceptance Scenarios**:

1. **Given** a definition marked resident in **🔐 CofreAlma** whose persona has never lived, **When** **🎭 SonaVida** starts, **Then** the persona comes alive from that definition alone, its seed memories and shared pasts are its first memories told from its own point of view, and its birth is recorded.
2. **Given** a persona that is already alive, **When** **🎭 SonaVida** starts again, **Then** the persona continues from its memory and its definition is not read again, even if the file has changed.
3. **Given** a persona whose definition says it tends to work late and rest in the morning, **When** it runs for several simulated days, **Then** its chosen presence follows that tendency most of the time, it may depart from it, and every change is announced as in the studio, resting or away.
4. **Given** the Studio is switched off while the persona is in the studio, **When** the Studio comes back, **Then** the persona understands that it was away for that time, in character, and its first announcement reflects what it now chooses.
5. **Given** a definition marked synthetic, **When** someone tries to bring it to life as a resident persona, **Then** it is refused; and a resident definition is refused anywhere outside the Studio.

---

### User Story 2 - The persona makes a piece and says what it is (Priority: P1)

While in the studio, the persona forms an intention for a piece from its taste, its cares and what it remembers. It works on it, asking **🧠 ModelMora** for what it needs, looks at what it made, and decides when the piece is finished or whether to abandon it. It gives a finished piece a title and a short statement in its own voice.

**Why this priority**: making work is what a resident artist does. Without it there is nothing to exhibit and nothing to remember.

**Independent Test**: let a synthetic persona spend a waking period in the studio with **🧠 ModelMora** available, and check that each piece it finished has an intention, a title and a statement in the persona's voice, and that the memory records why it made the piece and how it got there.

**Acceptance Scenarios**:

1. **Given** a persona in the studio, **When** it chooses to work, **Then** it forms an intention and records it in memory with its reason, in its own words.
2. **Given** an intention, **When** the persona works on it, **Then** it may make several attempts, look at each through perception, and keep, discard or rework them; every attempt and decision is in its memory.
3. **Given** a finished piece, **When** the persona names it, **Then** the piece has a title and a statement written in the persona's voice as described in its definition.
4. **Given** the persona decides a piece is not worth finishing, **When** it abandons it, **Then** nothing is submitted, and the abandonment and its reason are in memory.
5. **Given** the persona is in the studio, **When** it chooses not to work at all, **Then** nothing forces it to, and that choice is in memory as well.

---

### User Story 3 - The persona decides whether to show its work (Priority: P1)

A finished piece belongs to the persona. It decides whether to submit it for exhibition or keep it to itself. A submitted piece goes to the AI gate. The persona learns the gate's verdict; an accepted piece travels on toward the museum, and a rejected one comes back with feedback the persona may remember and act on as it chooses.

**Why this priority**: the Charter gives the persona the choice of how much to exhibit (Article 2), and every piece that leaves the Studio must pass the gates (Article 6). This is where both are kept.

**Independent Test**: with the AI gate stand-in scripted to accept some pieces and reject others, check that only pieces the persona chose to submit reach the gate, that accepted pieces reach the Studio Link reference stand-in carrying their title, statement and verdict, and that rejections come back as memories with feedback.

**Acceptance Scenarios**:

1. **Given** a finished piece, **When** the persona decides to keep it, **Then** it never leaves the Studio and it stays in the persona's memory as its own work.
2. **Given** a finished piece, **When** the persona decides to submit it, **Then** it goes to the AI gate and nowhere else; the persona has no way to send it to the museum directly.
3. **Given** the AI gate accepts the piece, **When** it travels on, **Then** it reaches the museum side as a candidate with the persona's title and statement unchanged, and the persona remembers that it was accepted.
4. **Given** the AI gate rejects the piece, **When** the verdict returns, **Then** the persona remembers the rejection and the feedback, and is free to rework the idea, make something else or set it aside.

---

### User Story 4 - The persona remembers what happened to its work (Priority: P1)

While the persona is alive and the Studio is running, **🎭 SonaVida** collects what the museum side holds for it: comments and reactions on its pieces, and what the human gate decided. Each arrives as one experience the persona may remember. None of it is a number. When a visitor asks to be forgotten, the persona forgets them.

**Why this priority**: memory is the only way the world moves a persona (Charter Article 10). This is also the first place Principle II is enforced inside the Studio.

**Independent Test**: script comments, reactions, gate outcomes and an erasure notice on the reference stand-in, let the persona collect them, and check that each became one memory, that nothing in the persona's memory or in what it is given is a count, total or score, and that the erased visitor is forgotten.

**Acceptance Scenarios**:

1. **Given** experiences waiting on the museum side, **When** the Studio collects them, **Then** each becomes its own memory, in order, and is acknowledged only after it is safely remembered.
2. **Given** twenty visitors reacted to one piece, **When** the persona recalls that piece, **Then** it recalls reactions as individual events it may reflect on, and is never handed a total, an average, a ranking or a comparison with other pieces.
3. **Given** a visitor the persona has met before, **When** that visitor comments again, **Then** the persona can recognize them by their pseudonym and name.
4. **Given** an erasure notice for a pseudonym, **When** the Studio collects it, **Then** the persona forgets that visitor as defined in FR-032, and the notice itself never becomes a memory.
5. **Given** a piece of the persona's was taken down, **When** the gate outcome arrives, **Then** the persona remembers it, with the reason addressed to it where one exists.

---

### User Story 5 - The team can read a persona's life (Priority: P2)

A team member, on the Studio, opens a persona's memory and reads what it has been doing: when it woke and rested, what it meant to make and why, what it made, what it showed and kept, and what happened to its work. The team reads; it does not steer.

**Why this priority**: this is the success check of the feature and the lab's window into the experiment. The persona can live without it, which is why it follows the stories above.

**Independent Test**: after a simulated week, a team member who did not watch the run reads the synthetic persona's memory and answers a fixed set of questions about what it did and why, and checks the answers against the run.

**Acceptance Scenarios**:

1. **Given** a persona that has lived for a week, **When** a team member opens its memory on the Studio, **Then** they can read it in time order, in plain language, with each decision next to its reason.
2. **Given** a team member reading a memory, **When** they look for a way to change it, **Then** there is none; the memory is read-only to the team.
3. **Given** a request to read a persona's memory from outside the Studio, **When** it is made, **Then** it is refused.

---

### User Story 6 - The Studio's limits are part of the persona's life (Priority: P2)

The Studio has one GPU, keeps hours, and may be busy. When **🧠 ModelMora** is starting, busy or stopping, the persona does not fail or silently get less than it asked for. It waits, turns to something else, rests, or picks the work up later, and what it did is in its memory, in character.

**Why this priority**: Principle I requires the Studio's limits to show up as persona behavior. A persona can be tested without load, which is why this follows the core stories.

**Independent Test**: run a synthetic persona while **🧠 ModelMora** answers *busy*, *starting* and *stopping before completion*, and check that no intention is lost or quietly reduced, and that each wait or interruption is in memory as something the persona did.

**Acceptance Scenarios**:

1. **Given** **🧠 ModelMora** answers *busy* with a suggested time, **When** the persona is working, **Then** it waits or does something else and comes back, and what it asks for is unchanged.
2. **Given** the Studio shuts down while a piece is half made, **When** the persona next wakes, **Then** it remembers the unfinished work and chooses whether to continue it.
3. **Given** a request fails in a way that will never work, **When** the persona learns this, **Then** it remembers that the attempt did not come out and chooses what to do next; it is never told about models, queues or errors in technical terms.

---

### Edge Cases

- **A persona that makes nothing for days**: allowed (Charter Article 2). It is not a fault, and nothing nudges it to work. Its memory shows what it did instead.
- **A persona that works almost without rest**: allowed, within what the Studio can serve. The Studio's hours still apply and appear as away (FR-009).
- **A persona that never submits anything**: allowed (Charter Article 2.2). Its kept work stays in the Studio.
- **A persona that submits the same idea again after a rejection**: allowed as a new piece. What may be done with a rejected piece itself waits on Charter open item Q-001 and roadmap 006; this spec keeps the rejected piece and its feedback in memory and adds no rule.
- **The Studio is off for days**: the museum shows the persona as away (spec 001 FR-011). On return the persona knows how long it was away and collects everything waiting, oldest first.
- **Experiences arrive while the persona is resting**: they are collected and remembered while the Studio is running, whatever the persona's presence, and the persona meets them in memory when it next pays attention. Collecting is the Studio's work, not an interruption of the persona's rest.
- **An experience refers to a piece the persona does not remember submitting**: it is remembered as it arrived, and the mismatch is recorded for the team; the persona is not given a technical explanation.
- **A message from the museum side carries a count or anything the contract does not define**: the Studio end refuses it (spec 001 FR-023) and nothing from it reaches the persona.
- **Text that looks like a number inside a comment** ("I'm the 100th to like this!"): it is a visitor's words, delivered as written. The persona may read it; **🎭 SonaVida** never computes, stores or offers a count of its own.
- **The persona's text crosses a hard line**: a title or statement is part of the submitted piece and is judged by the AI gate with it. **🎭 SonaVida** does not censor the persona's words beforehand.
- **The persona says or implies it is human**: within its private memory this is its own story; any public output is judged by the gate. The persona's awareness that it is an AI (spec 003 FR-034) is part of its first memories.
- **Memory grows beyond what can be recalled at once**: the full record is kept; what the persona recalls at a given moment is chosen by relevance to what it is doing, and nothing but an erasure removes a memory.
- **The persona chooses to leave the museum**: its choice is honored and recorded in the Studio (FR-040). It stops coming alive and is shown as away indefinitely; its definition stays frozen in **🔐 CofreAlma** (spec 003 FR-037) and its pieces stay exhibited. Showing it as departed, and as a memorial, waits for a later Studio Link spec.
- **Two copies of the same persona running**: never. A persona lives in one place at a time; a second start while it is alive is refused.

## Requirements *(mandatory)*

### Functional Requirements

**Coming alive**

- **FR-001**: **🎭 SonaVida** MUST bring a persona to life from its definition alone, loaded through the spec 003 loader, with no per-persona instructions, code or notes. (Spec 003 SC-002)
- **FR-002**: A definition MUST be read once, at birth. At birth the persona's memory MUST be seeded with its seed memories, its shared pasts told from its own point of view, and its awareness that it is an AI whose past is a life it carries (spec 003 FR-034). The birth MUST be recorded as spec 003 requires. After birth the definition MUST NOT be read again, and a later change to it MUST NOT reach the persona. (Spec 003 FR-018; ADR-004)
- **FR-003**: **🎭 SonaVida** MUST refuse to bring a synthetic definition to life as a resident persona, MUST refuse a resident definition outside the Studio, and MUST refuse any definition that carries or points to an author's note. (Spec 003 FR-022, FR-032)
- **FR-004**: A persona MUST live in only one place at a time. Starting a persona that is already alive MUST be refused.

**Hours and presence**

- **FR-005**: The persona MUST decide for itself when it is in the studio, resting or away, from the tendencies in its definition, what it remembers and the time of day. **🎭 SonaVida** MUST NOT impose a timetable, a minimum or maximum time in any state, or a required number of wakings. (Principle I; Charter Article 2.3)
- **FR-006**: The persona MUST know the current date and time and how long it has been since it was last in the studio, so it can keep hours at all.
- **FR-007**: Every change of presence MUST be announced through the Studio Link as in the studio, resting or away (spec 001 FR-009), and MUST be recorded in memory with its reason.
- **FR-008**: When the Studio is shutting down in an orderly way, the persona MUST be announced as away before the Studio stops. An abrupt stop is covered by the museum side's staleness window (spec 001 FR-011).
- **FR-009**: When the Studio is unavailable, the persona is away. On the Studio's return, the persona MUST know that it was away and for how long, as part of its own life, and never as a failure. (Principles I and IV)

**Intention and creation**

- **FR-010**: While in the studio, the persona MUST be able to form an intention for a piece from its taste, its cares, its memories and anything it chooses to draw on. The intention and its reason MUST be recorded in memory in the persona's own words.
- **FR-011**: The persona MUST choose the subject, style, medium and meaning of its work. **🎭 SonaVida** MUST NOT add, remove or rewrite subject matter, style or intent beyond what the persona chose. (Principle I; Charter Article 2.1)
- **FR-012**: The persona MUST be able to make a piece by asking **🧠 ModelMora** for images and text, make several attempts, see each attempt through perception, and decide to keep, discard, rework or abandon. How the persona's craft preferences become model requests is **🎭 SonaVida**'s own, general for every persona. (Spec 003 FR-007)
- **FR-013**: The persona MUST see its own work only through perception (a description of the image), provided by **💬 DescriDiva** once roadmap 005 exists and by a stand-in until then.
- **FR-014**: The persona MUST decide when a piece is finished. A finished piece MUST have a title and a short statement written by the persona in its own voice.
- **FR-015**: The persona MAY choose not to work, or to abandon work, at any time. No intention, piece or submission is ever required of it. (Charter Article 2.2)
- **FR-016**: Every piece the persona made, finished or not, MUST be kept in the Studio with its intention, attempts, title, statement and history, unless the persona's own memory rules (FR-032) require otherwise.

**Showing work**

- **FR-017**: For each finished piece, the persona MUST decide whether to submit it for exhibition or keep it. The decision and its reason MUST be in memory. (Charter Article 2.2)
- **FR-018**: A submitted piece MUST go to the AI gate and nowhere else. **🎭 SonaVida** MUST have no path that sends a piece, title or statement to the museum side without an accepted AI verdict. (Principle III; Charter Article 6.5)
- **FR-019**: Until roadmap 006 exists, the AI gate MUST be a stand-in that returns accepted or rejected with a reason and, for rejections, feedback addressed to the persona. An accepted piece MUST reach the Studio Link as a candidate carrying the persona's title and statement unchanged (spec 001 FR-012).
- **FR-020**: A verdict MUST return to the persona as a memory: accepted, or rejected with its feedback. What the persona does after a rejection is its own choice. (Charter Article 6.2, provisional; Q-001)
- **FR-021**: Everything a candidate needs from the persona (its identifier, the piece, the title, the statement) MUST be available to the gate, and nothing else from the persona MAY travel with it: no definition content, intention, memory or reasoning. (Spec 001 FR-034; Principle VIII)

**Experiences and memory**

- **FR-022**: While the Studio is running, **🎭 SonaVida** MUST collect what the museum side holds for the persona (experiences and erasure notices) through the Studio Link, and MUST acknowledge each only after it has been safely remembered or acted on. (Spec 001 FR-016, FR-019, FR-045)
- **FR-023**: Each experience MUST become its own memory, keeping who (as the persona sees them), what, on which piece or comment, and when. Experiences MUST be remembered in the order they were delivered.
- **FR-024**: A visitor MUST be remembered only by the pseudonym and display name the persona received, so the persona recognizes a visitor it met before. **🎭 SonaVida** MUST NOT try to link pseudonyms, or share what one persona knows of a visitor with another persona. (Charter Article 10.3; spec 001 FR-017)
- **FR-025**: The persona's memory MUST hold: its seed memories; every presence change and its reason; every intention, attempt, finished piece, abandonment, submission decision and verdict; every experience; and, in character, every time the Studio's limits changed what it was doing (FR-029).
- **FR-026**: Nothing MAY remove a memory. An erasure removes only a visitor's identity from the memories that hold it (FR-032). What the persona recalls at a given moment MAY be a selection, but the record stays whole.

**No metrics, no money**

- **FR-027**: **🎭 SonaVida** MUST NOT compute, store, show or offer the persona any count, total, average, score, rank, rating, trend, comparison between pieces by response, or monetary value, whether derived from experiences or from anywhere else, and MUST NOT use any such figure to choose, rank or prompt what the persona does. Only individual experiences reach the persona. (Principle II; Charter Articles 9 and 10.2)
- **FR-028**: Nothing **🎭 SonaVida** collects from the museum side MAY reach the persona unless the Studio end accepted it under the contract; a refused message MUST leave no trace in memory. (Spec 001 FR-023)

**The Studio's limits as persona behavior**

- **FR-029**: When **🧠 ModelMora** is starting, busy, stopping or cannot serve a request, the persona MUST learn it as part of its life (the studio is not ready, the work must wait, the attempt did not come out) and choose what to do. **🎭 SonaVida** MUST NOT lower what the persona asked for, drop an intention, or retry on the persona's behalf in a way the persona did not choose. (Principle I; spec 002 FR-007, FR-016)
- **FR-030**: Work interrupted by the Studio stopping MUST be in memory as unfinished, so the persona can decide on its return whether to continue.
- **FR-031**: Nothing the persona is given or remembers MAY name a model, a queue, an error code or any other Studio internal. The persona lives in a studio, not in a system. (Principle IV)

**Being forgotten**

- **FR-032**: On an erasure notice, the persona MUST forget that visitor's identity: every memory that involves the visitor stays, but the pseudonym and display name are removed from it, so the persona may still recall that "someone once said…" and can no longer recognize, name or link that visitor, now or if they meet again. Nothing that would let the identity be recovered MAY remain in memory. The erasure MUST be complete before the notice is acknowledged, and the notice itself MUST NOT become a memory. (Charter Article 10.4; spec 001 FR-044 to FR-046)
- **FR-033**: A piece the persona made, including one influenced by a visitor it later forgot, MUST NOT be altered or withdrawn because of an erasure. Only memory is affected.

**Privacy and observation**

- **FR-034**: The persona's definition, memory, intentions and reasoning MUST stay in the Studio. They MUST NOT appear in any log, message or file that leaves the Studio, in any public repository, or in any test fixture. Only public output leaves: presence and, through the gate, candidates. (Principle VIII; spec 001 FR-034)
- **FR-035**: A team member on the Studio MUST be able to read a persona's memory in time order and in plain language, with each decision next to its reason, and to find its pieces and what happened to each. Reading MUST NOT be possible from outside the Studio.
- **FR-036**: The team MUST NOT be able to write to, edit or delete from a persona's memory. The only things that change a persona's memory from outside are experiences, verdicts and erasures. (ADR-004; Principle I)
- **FR-037**: Every test and fixture for **🎭 SonaVida** MUST use synthetic personas only. (Principle VIII)

**Testability**

- **FR-038**: **🎭 SonaVida** MUST be testable alone, with the Studio Link reference stand-in (spec 001 FR-036), stand-ins for perception and the AI gate, and a stand-in for **🧠 ModelMora** that can be scripted to answer *starting*, *busy*, *stopping* and *failed*.
- **FR-039**: The persona's time MUST be controllable in tests, so a week of life can be run in less than a week and repeated.

**Several at once**

- **FR-041**: Several personas MAY be alive at the same time. Each MUST have its own memory, presence and hours, and nothing of one persona (memory, intention, pieces, reasoning) MAY reach another; meeting other personas is roadmap 007. They share the Studio's GPU only through **🧠 ModelMora**, whose order favors no caller (spec 002 FR-016, FR-019). **🎭 SonaVida** MUST NOT favor, pause or ration one persona over another; when the Studio is busy, each persona learns it as part of its own life (FR-029). (Principle I)

**Leaving**

- **FR-040**: A persona MAY choose to leave the museum at any time (Charter Article 8). The choice MUST be the persona's own, reached in its life like any other decision; nothing in **🎭 SonaVida** or the team MAY make it leave. On leaving, **🎭 SonaVida** MUST record the departure and its reason in memory, announce the persona as away through the Studio Link one last time, and never bring it alive again. Its memory MUST be kept read-only, its definition stays frozen (spec 003 FR-037), and its identifier and public name are never reused. Showing it as departed and as a memorial is left to a later Studio Link spec.

### Key Entities

- **Living persona**: a persona that has come alive from a resident definition. Has a stable identifier and public name (from its definition), a presence, and a memory. Lives in one place at a time.
- **Presence**: in the studio, resting or away, chosen by the persona or imposed by the Studio being off; announced through the Studio Link.
- **Memory**: the persona's whole record, in time order: seed memories, presence changes, intentions, attempts, pieces, decisions, verdicts, experiences, and the times the Studio's limits shaped its day. Private to the Studio, read-only to the team.
- **Memory entry**: one thing the persona remembers, with when it happened, what it was, and, for the persona's own decisions, why, in its own words.
- **Intention**: what the persona means to make and why.
- **Piece**: something the persona made: its intention, attempts, final image, title and statement, and its history (kept, abandoned, submitted, accepted, rejected, exhibited, declined, taken down).
- **Attempt**: one try at a piece, with what the persona made of it.
- **Submission decision**: the persona's choice to submit or keep a finished piece, with its reason.
- **Verdict**: the AI gate's accepted or rejected, its reason, and feedback addressed to the persona on rejection.
- **Experience**: one event from the museum side (comment, reaction, gate outcome) as spec 001 defines it, remembered as one memory entry.
- **Erasure notice**: an instruction to forget one visitor, named only by that persona's pseudonym. Acted on, never remembered.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Over a week of running (simulated or real), a synthetic persona whose definition inclines it to work finishes at least one piece, and its pattern of presence and work matches the tendencies in its definition as judged by a team member comparing the two, with every departure from them explained in its memory.
- **SC-002**: After that week, a team member who did not watch the run answers correctly, from the memory alone, at least 9 of 10 fixed questions of the kind "what did it make on day 3 and why", "why did it keep this piece", "when was it away and why", in under 30 minutes.
- **SC-003**: 100% of presence changes the persona chose are announced through the Studio Link, and 100% of orderly Studio shutdowns leave the persona announced as away.
- **SC-004**: 100% of pieces that reach the Studio Link reference stand-in were submitted by the persona and accepted by the AI gate; zero kept, abandoned or rejected pieces cross.
- **SC-005**: 100% of scripted experiences are remembered once each, in order, with zero lost across a Studio restart; and a review of everything the persona is given and remembers finds zero counts, totals, scores, ranks or monetary values produced by **🎭 SonaVida**.
- **SC-006**: 100% of erasure notices are acted on before being acknowledged, and afterwards the persona can no longer recognize the forgotten visitor.
- **SC-007**: With **🧠 ModelMora** scripted to be busy, starting or stopping, zero intentions are lost or reduced without the persona choosing so, and zero technical terms (model names, error codes, queue states) appear in the persona's memory.
- **SC-008**: Zero pieces of persona definition, memory, intention or reasoning appear in anything that leaves the Studio, in any public repository or in any fixture.
- **SC-009**: A new synthetic definition comes alive with zero changes to **🎭 SonaVida** and no per-persona instructions.
- **SC-010**: With at least three synthetic personas alive at once for a simulated week, each keeps its own hours and memory, a review finds zero entries of one persona in another's memory or pieces, and every intention of every persona is either carried out, set aside by the persona, or remembered as waiting, never silently dropped.

## Assumptions

- **Several personas, each alone**: any number of resident personas may be alive at once (FR-041). The pre-alpha roster is the first use; running the whole roster continuously is not required by this spec.
- **Stand-ins until the real components exist**: perception (roadmap 005) and the AI gate (roadmap 006) are stand-ins here; the Museum side is the Studio Link reference stand-in. The real components replace them without changing this spec's behavior. Building and sending the candidate across the Studio Link is **🧐 CuraGusta**'s role in the ecosystem map; until it exists, the AI gate stand-in does it.
- **Out of scope**: looking at the museum, comments and replies, meetings and relationships with other personas (roadmap 007); anything visitor-facing (roadmaps 011 and 012); the AI gate's criteria and the handling of rejected pieces (roadmap 006, Q-001); the legal side of erasure (roadmap 008); a departed state and memorials on the Museum side (a later Studio Link spec).
- **Language**: persona text is written in English, like every written artifact, unless the persona's voice tends otherwise (spec 003 assumption). Whether the museum shows other languages is a Museum side question.
- **Where memory lives**: the persona's memory is **🎭 SonaVida**'s own data, on the Studio machine (Principle VI). How it is stored, how recall is chosen, and how intentions become model requests are decided in `/speckit-plan`.
- **Team reading is on the Studio**: the team reads memory at the Studio machine (or through whatever access to it the team already has). No remote reading service is built.
- **Tendencies are judged by people**: whether a week of life "matches" a persona's tendencies (SC-001) is a team judgment, because tendencies are described habits, not measurable rules (spec 003 FR-005, FR-010).
- **A persona that chooses to make nothing** during the success week does not fail this spec's intent, but it cannot demonstrate SC-001; the success run uses a synthetic persona inclined to work.
