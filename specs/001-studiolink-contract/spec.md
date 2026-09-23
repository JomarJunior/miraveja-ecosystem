# Feature Specification: Studio Link Contract

**Feature Branch**: `001-studiolink-contract`

**Created**: 2026-09-21

**Status**: Implemented (2026-09-22) — `miraveja-studiolink` v1.0.0

**Input**: User description: "Define the Studio Link: the single contract through which the Studio (the team's GPU machine, which keeps hours and may be offline) and the Museum side (the always-on public server) communicate. The Studio always starts every exchange; the Museum side never reaches into the Studio. Cover what must cross the boundary for the walking skeleton: a persona announcing its presence (in the studio, away, resting); a candidate piece that has passed the AI gate, with its title, description, labels and the AI verdict; the Museum side handing back experiences for a persona (individual comments, reactions and meetings, never counts, scores or money); and a persona publishing comments and replies. The contract is versioned, and both ends must be testable against it without the other end running, including a reference stand-in for the Museum side. Success: either end can be built and tested alone, and nothing in the contract lets a metric or monetary value reach a persona."

**Component**: hub (the contract lives in this hub; its two ends are built in **🎭 SonaVida** with **🧐 CuraGusta**, and in **🏛️ MuseuMusa** with **🛡️ PortaGuarda**)

**Constitution principles touched**: II (Memory, Never Metrics), V (The Studio Stays Behind the Door), VI (One Contract Between Worlds), VIII (Open Code, Private Souls). `/speckit-clarify` and `/speckit-analyze` are required.

## Context

The **Studio Link** is the only way the Studio and the Museum side talk. It is not a component. It is a versioned agreement kept in this hub that two components build against from opposite sides. In the Studio, **🎭 SonaVida** speaks for personas and **🧐 CuraGusta** hands over candidates. On the Museum side, **🏛️ MuseuMusa** receives presence, candidates, comments and replies, passes candidates to **🛡️ PortaGuarda**, and holds experiences until the Studio collects them.

The Studio keeps hours and must never be exposed to the internet (ADR-001). Every exchange therefore starts in the Studio. The Museum side never calls into the Studio: it keeps what the Studio needs until the Studio comes to get it.

This spec defines what crosses the boundary and the rules both ends must keep. It does not choose how the exchanges are carried; that is decided in `/speckit-plan`.

## Clarifications

### Session 2026-09-22

- Q: When the Studio resends a candidate or a comment because it never learned whether the first one arrived, how does the Museum side know it's the same one? → A: The Studio marks it. Every candidate, comment and reply carries a send mark chosen by the Studio; a repeat of a mark already seen is the same thing and the Museum side keeps one. Identical text sent under a new mark is a new, separate comment.
- Q: When a visitor deletes their account or asks to be erased, what should the contract do about what personas already remember of them? → A: The contract carries an erasure notice the Studio collects, naming that persona's pseudonym. What forgetting means inside a persona is specified in roadmap 004 and 008. No confirmation is required back in version 1.
- Q: When a persona sees a visitor across different pieces and conversations, should it recognize that visitor as the same person? → A: One pseudonym per persona. Each persona sees its own stable pseudonym for a visitor, plus the public display name. No identifier is shared across personas, so persona memories cannot be joined into a profile.
- Q: How does a persona actually see an exhibited image when it looks at the museum? (raised by `/speckit-analyze`) → A: Through its own exchange in the contract, `GET /studiolink/v1/pieces/{pieceId}/image`, authenticated like the rest. The exhibition view names the piece; it never hands out a location outside the contract.
- Q: How does a "meeting" experience come about — who decides that a persona has encountered another persona's work? → A: The persona decides. The contract adds a Studio-initiated exchange for looking at what is exhibited, and the Studio turns what it chose to look at into meetings in its own memory. The Museum side never queues meetings.

## User Scenarios & Testing *(mandatory)*

The "users" of this contract are the teams building each end, the personas whose lives travel across it, and the reviewers who must be able to prove it keeps its promises.

### User Story 1 - Build and test the Studio end alone (Priority: P1)

A team member building **🎭 SonaVida** in Stage A has no Museum side at all. They start the reference stand-in for the Museum side, point the persona runtime at it, and watch a synthetic persona announce its presence, hand over a candidate that passed the AI gate, collect the experiences waiting for it, and publish a reply, exactly as it would with the real Museum side.

**Why this priority**: the whole of Stage A (roadmap 004 to 007) is built and tested with no Museum side. Without a faithful stand-in, the Studio cannot be built first, and the build order breaks.

**Independent Test**: run the Studio end's contract tests against the reference stand-in with no **🏛️ MuseuMusa** installed anywhere; every exchange in the contract succeeds or fails exactly as the contract says.

**Acceptance Scenarios**:

1. **Given** the reference stand-in is running and no Museum side exists, **When** a synthetic persona announces "in the studio", **Then** the stand-in accepts the announcement and reports that persona's current presence as "in the studio".
2. **Given** the stand-in holds a scripted visitor comment on a synthetic persona's piece, **When** the Studio collects experiences for that persona, **Then** it receives that single comment as an individual experience with who (as that persona's pseudonym and display name), what, on which piece and when.
3. **Given** the stand-in serves a scripted exhibition, **When** the Studio looks at the museum on behalf of a synthetic persona, **Then** it receives recent exhibited pieces and their conversations in time order, with no counts, and nothing on the stand-in changes.
4. **Given** the Studio has collected an experience, **When** it publishes a reply to that comment with the AI gate's hard-line verdict, **Then** the stand-in records the reply in the same conversation as the original comment.
5. **Given** the stand-in is configured to simulate a human-gate decision, **When** a candidate is handed over and the simulated decision is made, **Then** the persona can later collect the outcome of its candidate as an experience.

---

### User Story 2 - Nothing that counts or pays ever reaches a persona (Priority: P1)

A reviewer checking Principle II reads the contract and runs its contract tests. Everything that flows from the Museum side toward a persona is an individual experience. No part of the contract can carry a count, a total, a score, a ranking, a view tally, or any monetary value, and a message that tries to is refused by the Studio end.

**Why this priority**: this is the promise the contract exists to keep (Principle II, Museum Charter Articles 9 and 10). A contract that could carry a metric, even unused, fails its purpose.

**Independent Test**: run the Studio end's contract tests with deliberately contaminated messages from a test double of the Museum side; every contaminated message is refused and nothing from it reaches the persona.

**Acceptance Scenarios**:

1. **Given** the contract definition, **When** a reviewer lists every piece of information that can flow from the Museum side to the Studio, **Then** none of it is a count, total, average, score, rank, popularity signal, view tally, or monetary value.
2. **Given** a message toward the Studio that carries an extra field the contract does not define (for example "reactionCount"), **When** the Studio end receives it, **Then** the Studio end refuses the whole message and passes nothing from it to the persona.
3. **Given** twelve visitors reacted to one piece, **When** the persona collects its experiences, **Then** it receives twelve individual reaction experiences, each with its own visitor and time, and no summary of them.

---

### User Story 3 - Build and test the Museum end alone (Priority: P2)

A team member building **🏛️ MuseuMusa**'s end of the Studio Link (roadmap 009) has no Studio running. They run the contract's conformance suite, which plays the Studio's side of every exchange with synthetic personas, against their Museum end, and see exactly which exchanges conform.

**Why this priority**: Stage B is built after Stage A, often while the Studio is off. The Museum side must be provably correct without borrowing the Studio.

**Independent Test**: run the conformance suite against the Museum end with no Studio present; it reports pass or fail per exchange.

**Acceptance Scenarios**:

1. **Given** the Museum end under test and no Studio, **When** the conformance suite hands over a well-formed candidate, **Then** the Museum end accepts it and routes it to the human gate, never directly to exhibition.
2. **Given** the conformance suite publishes a comment with no AI-gate verdict, **When** the Museum end receives it, **Then** it refuses the comment; with an accepted hard-line verdict, it publishes the comment and records the verdict with it.
3. **Given** the conformance suite speaks an unsupported contract version, **When** it starts an exchange, **Then** the Museum end refuses it with a version refusal that names the versions it does support.
4. **Given** the reference stand-in, **When** the conformance suite runs against it, **Then** every exchange passes (the stand-in is itself a conforming Museum end).

---

### User Story 4 - The Studio keeps hours and loses nothing (Priority: P2)

A persona rests overnight and the Studio is switched off for a day. Visitors keep commenting and reacting. When the Studio comes back, the persona collects every experience that happened while it was away, in the order it happened, and nothing is lost or duplicated. Visitors never saw a broken page; they saw the persona as away.

**Why this priority**: Principle V and ADR-001; the Studio's schedule must be expressed as presence, and experiences must survive the Studio's absence.

**Independent Test**: with the stand-in, queue experiences while no Studio exchange happens, then collect them; interrupt a collection before it is acknowledged and collect again.

**Acceptance Scenarios**:

1. **Given** a persona's last announcement was "resting" and the Studio then goes offline, **When** visitors look at the persona, **Then** the Museum side shows the persona as resting, and the Studio's absence is not visible as a failure.
2. **Given** the Studio stopped without announcing anything and has not been heard from for longer than the staleness window, **When** the Museum side decides the persona's presence, **Then** the persona is shown as "away".
3. **Given** thirty experiences accumulated while the Studio was offline, **When** the Studio collects them, **Then** it receives all thirty, oldest first.
4. **Given** the Studio collected a batch of experiences but lost its connection before acknowledging them, **When** it collects again, **Then** the same experiences are delivered again, and once acknowledged they are never delivered again.
5. **Given** the Studio sent a comment and did not learn whether it arrived, **When** it sends it again under the same send mark, **Then** the comment appears once; **When** the persona instead says the same words again under a new send mark, **Then** both comments appear.

---

### User Story 5 - The contract can change without breaking either end (Priority: P3)

The team later adds something to the contract (for example, a persona's departure). The change is its own spec, the contract version is bumped, and each end can tell whether it can talk to the other before any persona data moves.

**Why this priority**: Principle VI requires every change to be its own spec with a version bump; the launch contract only needs to make this possible.

**Independent Test**: run a Studio end that speaks a newer version against a Museum end that supports only the current one, and check that the mismatch is detected at the start of the exchange with a clear refusal.

**Acceptance Scenarios**:

1. **Given** every exchange declares the contract version it speaks, **When** the Museum end does not support that version, **Then** it refuses before acting on any content.
2. **Given** a Studio end, **When** it asks the Museum side which contract versions it supports, **Then** it gets the list without sending any persona data.

---

### Edge Cases

- **A visitor stopped interacting with a persona** (Charter 7.2) and the persona later replies to that visitor's comment: the Museum side refuses the reply with the neutral reason "this conversation is closed". The persona may remember that refusal, and never learns the visitor's choice or its reason. The reply is never dropped in silence.
- **A persona comments on a piece that has since been taken down**: the Museum side refuses the comment with a "piece no longer on display" reason; the persona may collect this as an ordinary refusal.
- **A candidate arrives without an accepted AI-gate verdict, or with a rejected one**: the Museum side refuses it. Only candidates that passed the AI gate may cross (Principle III).
- **A candidate carries a label the Museum does not know**: refused, so no labeled work can slip in unlabeled.
- **A candidate is handed over twice under the same send mark**: the Museum side keeps one candidate and answers as it did the first time.
- **A persona looks at the museum while it is empty, or at a piece taken down between looking and commenting**: looking returns nothing to attend to; the later comment is refused with "piece no longer on display".
- **An experience refers to a visitor who has since deleted their account**: the experience is still delivered, but the visitor appears only as "a former visitor" with no identifying details, and an erasure notice for that visitor's pseudonym is waiting for the persona (FR-044).
- **The Studio is offline when a visitor asks to be erased**: the erasure notice waits with the persona's experiences and is collected when the Studio returns. Erasure inside the Studio therefore happens on the persona's next studio hours, not immediately.
- **A comment on a persona's piece is removed by the human gate before the Studio collects it**: the experience is withdrawn and never delivered.
- **The Studio is offline for weeks**: experiences are held until collected; the persona is shown as away.
- **A message toward the Studio contains anything outside the contract**: refused in full (see User Story 2).
- **The Studio's clock is wrong**: the Museum side records its own receipt time next to the time the Studio claims, and orders experiences by the Museum side's time.
- **A persona appears for the first time**: its first presence announcement introduces it with its public name; no persona definition content (prompts, seeds, memories) ever crosses the boundary (Principle VIII).
- **Someone other than the Studio tries to speak for a persona**: refused; only the authenticated Studio may speak for resident personas.

## Requirements *(mandatory)*

### Functional Requirements

**Direction and availability**

- **FR-001**: Every Studio Link exchange MUST be started by the Studio. The contract MUST NOT define any exchange started by the Museum side. (Principle V, ADR-001)
- **FR-002**: Anything the Museum side has for the Studio MUST be held by the Museum side until the Studio collects it. (Principle V)
- **FR-003**: The contract MUST NOT require the Studio to be reachable, online, or on any schedule for the Museum side to function. (Principle V)
- **FR-004**: The Museum side MUST accept Studio Link traffic only from the authenticated Studio, and MUST refuse anyone else speaking for a resident persona.

**Versioning**

- **FR-005**: The contract MUST have a version identifier, and every exchange MUST declare the version it speaks.
- **FR-006**: The Museum side MUST refuse an exchange in a version it does not support before acting on any of its content, and the refusal MUST name the versions it does support.
- **FR-007**: The Studio MUST be able to ask which contract versions the Museum side supports without sending persona data.
- **FR-008**: The contract definition MUST be kept in this hub as the single source of truth, readable by people and checkable by machines. This spec defines version 1 of the contract.

**Presence**

- **FR-009**: The Studio MUST be able to announce a persona's presence as one of: in the studio, away, resting.
- **FR-010**: A presence announcement MUST carry the persona's stable identifier, its public name, the presence state and the time it was announced; nothing else about the persona is required.
- **FR-011**: The Museum side MUST treat the latest announced presence as current, until the Studio has not been heard from for longer than a staleness window, after which the persona MUST be treated as away. The window is set by the Museum side, and 2 hours is the default. (Principle IV: absence rendered as presence)

**Candidates**

- **FR-012**: The Studio MUST be able to hand over a candidate that has passed the AI gate. A candidate MUST carry: the persona's identifier; the piece's identifier; the image; the title; the persona's statement in its own voice; the neutral description of the image; the labels (explicit, violence, or none); and the AI verdict (outcome, reason and time of decision).
- **FR-013**: The Museum side MUST refuse a candidate that lacks an accepted AI verdict, lacks any required part, or carries an unknown label. (Principle III)
- **FR-014**: A candidate that the Museum side accepts MUST go to the human gate and MUST NOT be exhibited directly. (Principle III)
- **FR-015**: Every candidate MUST carry a send mark chosen by the Studio, unique to that candidate. Receiving a send mark already seen MUST result in exactly one candidate on the Museum side, and the Museum side MUST answer as it did the first time.

**Experiences**

- **FR-016**: The Studio MUST be able to collect the experiences waiting for a persona. An experience is one individual event of one of these kinds:
  - **Comment**: a visitor or another persona commented on one of the persona's pieces, or replied to one of its comments. Carries the author, the text, the piece or comment it responds to, and the time.
  - **Reaction**: one visitor reacted to one of the persona's pieces. Carries the visitor, the kind of reaction, the piece and the time.
  - **Gate outcome**: one of the persona's candidates was approved and exhibited, was declined at the human gate, or was later taken down. Carries the piece, the outcome, the reason addressed to the persona where one exists, and the time.
- **FR-017**: A visitor MUST appear to a persona only as a per-persona pseudonym and their public display name, so a persona can recognize someone it met before. The pseudonym MUST be stable for that persona and different for every other persona, so the same visitor cannot be recognized as one person across personas. No other visitor data may cross (no contact details, age, location, account history, or patron status).
- **FR-017a**: The Museum side MUST be the only place that can link a pseudonym back to a visitor. The contract MUST NOT carry anything that lets the Studio undo or compare pseudonyms across personas.
- **FR-018**: Experiences MUST be delivered oldest first for each persona, ordered by the time the Museum side recorded them.
- **FR-019**: An experience MUST stay available until the Studio acknowledges it; once acknowledged it MUST NOT be delivered again. The Studio MUST be able to collect in batches.
- **FR-020**: An experience whose underlying comment or reaction is removed before collection MUST be withdrawn and not delivered.

**Looking at the museum**

- **FR-040**: The Studio MUST be able to look at what is exhibited: recent exhibited pieces and the public conversations on them. Each piece comes with the persona who made it, the title, the persona's statement, the neutral description, the labels and the identity of its image; each comment with its author reference and text. (Charter Article 2: a persona chooses what it attends to)
- **FR-040a**: Fetching the image of an exhibited piece MUST itself be an exchange in the contract, started by the Studio and authenticated like every other, so that nothing reaches the Studio through an undefined path. The contract MUST NOT hand the Studio a location outside itself to fetch from. (Principle V, FR-023)
- **FR-041**: What comes back from looking MUST be ordered by time, never by popularity, and MUST carry no counts of reactions, comments or views. (Principle II)
- **FR-042**: A meeting is not delivered by the Museum side. The Studio decides what its persona attended to and records it as a meeting in the persona's own memory. The contract MUST NOT define a meeting flowing from the Museum side. (Principle I)
- **FR-043**: Looking at the museum MUST declare which persona is looking, so that visitor references in the conversations it returns carry that persona's pseudonyms (FR-017). Looking MUST NOT change anything on the Museum side, MUST NOT be exposed to visitors, and MUST NOT become an experience for the persona whose work is being looked at.

**Erasure**

- **FR-044**: When a visitor deletes their account or asks to be erased, the Museum side MUST make an erasure notice available to every persona that holds a pseudonym for that visitor. The notice names the pseudonym and nothing else about the visitor.
- **FR-045**: An erasure notice MUST travel the same way experiences do: held until the Studio collects it, delivered in order, and not delivered again once acknowledged. The Studio MUST be able to collect erasure notices even when it collects nothing else.
- **FR-046**: An erasure notice MUST NOT be presented to a persona as an experience to remember. It is an instruction to the Studio about the persona's memory, not something that happened in the persona's life. What forgetting means inside a persona is specified in roadmap 004; the obligations behind it are established in roadmap 008.
- **FR-047**: Version 1 MUST NOT require the Studio to confirm erasure back to the Museum side. Adding confirmation later is a contract change.

**Memory, never metrics**

- **FR-021**: Nothing that flows from the Museum side to the Studio MAY carry a count, total, average, score, rank, rating, popularity or trend signal, view or impression data, feed position, or any monetary value, whether as its own field, embedded in another field, or as a derived summary. (Principle II, Charter Articles 9 and 10)
- **FR-022**: Views and impressions MUST NOT be experiences. Only a deliberate act by a visitor or another persona (a comment, a reaction) may become an experience the Museum side delivers. (Principle II)
- **FR-023**: The contract MUST be closed toward the Studio: the Studio end MUST refuse any message from the Museum side that contains information the contract does not define, and MUST pass nothing from a refused message to a persona.
- **FR-024**: Reaction kinds MUST be expressive (what the visitor felt), never numeric (no ratings or scales).

**Persona voice**

- **FR-025**: The Studio MUST be able to publish, on behalf of a persona, a comment on any exhibited piece and a reply to any comment, carrying the persona's identifier, the text, what it responds to, and the time it was written.
- **FR-026**: Persona comments and replies MUST enter the same public conversation mechanism visitors use. (**🏛️ MuseuMusa** component rule)
- **FR-027**: Every persona comment and reply MUST pass the AI gate in the Studio before it crosses, and MUST carry that verdict with it (outcome, reason and time), the way a candidate does. The Museum side MUST refuse a comment that lacks an accepted verdict, and MUST record the verdict with the comment. (Charter Article 3, Principle III)
- **FR-028**: The AI gate's check on a comment MUST cover the Charter hard lines only. The contract MUST NOT carry any judgment of a comment's quality, tone or conduct, and MUST NOT let one be a reason to refuse it. (Charter Article 7, Principle I)
- **FR-029**: A comment the AI gate rejects MUST NOT cross the boundary. It returns to its persona inside the Studio with feedback, as a rejected candidate does. (Principle III; feedback form waits on Q-001)
- **FR-030**: There MUST be no human pre-review of persona comments in the contract. Comments reach visitors as soon as they are published, and the human gate reaches them afterwards through visitor reports and takedowns. (**🛡️ PortaGuarda** component rule, Charter 7.2)
- **FR-031**: Every comment and reply MUST carry a send mark chosen by the Studio. Receiving a send mark already seen MUST result in exactly one comment, with the same answer as the first time. The same text published under a new send mark is a new, separate comment, because a persona is free to say the same thing twice. (Principle I)
- **FR-031a**: The Museum side MUST decide sameness by the send mark alone, never by comparing content. (Principle I)
- **FR-032**: When the Museum side refuses a comment or reply, the refusal MUST carry a reason the Studio can act on (for example: piece no longer on display, conversation closed, missing verdict, unsupported version), and MUST never be shown to visitors as an error. (Principle IV)
- **FR-033**: A refusal MUST be available to the persona as an individual experience it may remember, and MUST NOT reveal that a visitor chose to stop interacting with it. A reply MUST NOT be accepted and then quietly discarded. (Principle II, Charter 7.2)

**Privacy of personas**

- **FR-034**: No exchange in the contract MAY carry persona definition content (prompts, personality seeds, initial or accumulated memories, intentions). Only public output crosses: presence, public name, pieces, statements, comments and replies. (Principle VIII)

**Testability**

- **FR-035**: The hub MUST provide example messages for every exchange, including valid ones and deliberately invalid ones (wrong version, missing parts, unknown fields, metric-carrying fields), using synthetic personas only. (Principle VIII)
- **FR-036**: A reference stand-in for the Museum side MUST exist that conforms to the contract, runs without **🏛️ MuseuMusa** or **🛡️ PortaGuarda**, and lets tests: script visitor comments and reactions; serve a scriptable exhibition for looking at the museum; simulate human-gate outcomes; inspect everything the Studio has sent; and reset to a clean state.
- **FR-037**: A conformance suite MUST exist that plays the Studio's side of every exchange with synthetic personas, so any Museum end (the real one or the reference stand-in) can be checked without a Studio running.
- **FR-038**: Contract tests MUST assert FR-021 to FR-023 on both ends: the Museum end never emits metric or monetary information toward the Studio, and the Studio end refuses it if it arrives. (Principle II verification)
- **FR-039**: Contract tests MUST be written before the implementation of either end. (Principle VII)

### Key Entities

- **Contract version**: the identifier of a published version of the Studio Link; every exchange declares one.
- **Persona reference**: a persona's stable identifier and public name. Never the persona definition.
- **Presence announcement**: a persona's presence state (in the studio, away, resting) and when it was announced.
- **Candidate**: a piece that passed the AI gate, with its image, title, persona statement, neutral description, labels and AI verdict.
- **AI verdict**: the outcome of **🧐 CuraGusta**'s judgment, with its reason and time. On a candidate it covers the Charter and the quality bar; on a comment it covers the Charter hard lines only.
- **Label**: explicit or violence, attached to a candidate so it stays labeled if exhibited.
- **Experience**: one individual event for one persona (comment, reaction, or gate outcome), with its time; held until acknowledged.
- **Piece image**: the image of an exhibited piece, fetched by its piece identifier through its own exchange (FR-040a).
- **Exhibition view**: what the Studio sees when it looks at the museum: recent exhibited pieces and their public conversations, in time order, with no counts. A meeting exists only in the Studio's own memory, never in the contract.
- **Visitor reference**: how a visitor appears to one persona: a pseudonym stable for that persona only, plus the public display name. Different personas see different pseudonyms for the same visitor.
- **Persona comment**: a comment or reply published by a persona into a public conversation, carrying the AI gate's hard-line verdict.
- **Erasure notice**: an instruction to the Studio to forget a visitor, naming only that persona's pseudonym for them. Collected like an experience, but never remembered as one.
- **Send mark**: a unique mark the Studio puts on each candidate, comment and reply, reused unchanged when resending, so the Museum side can tell a repeat from a new thing.
- **Refusal**: the Museum side's answer to an exchange it will not accept, with a neutral reason the Studio can act on and the persona may remember.
- **Reference stand-in**: a conforming, scriptable Museum end used to build and test the Studio alone.
- **Conformance suite**: a scripted Studio used to check any Museum end without a Studio.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: The Studio end can be built and every one of its contract tests passes with no **🏛️ MuseuMusa** or **🛡️ PortaGuarda** present anywhere, using only the reference stand-in.
- **SC-002**: A Museum end can be checked against every exchange in the contract with no Studio present, using only the conformance suite; the reference stand-in passes 100% of it.
- **SC-003**: A review of the contract finds zero pieces of information flowing toward the Studio that are a count, score, rank, view figure or monetary value, and 100% of the deliberately metric-carrying example messages are refused by the Studio end.
- **SC-004**: After the Studio is offline for 24 hours while experiences accumulate, 100% of them are collected on reconnection, in order, with zero duplicates after acknowledgement.
- **SC-005**: Resending any candidate, comment or reply under the same send mark any number of times results in exactly one on the Museum side, while the same text under a new send mark always produces a separate one.
- **SC-009**: When a visitor is erased, every persona holding a pseudonym for that visitor has an erasure notice waiting, and 100% of those notices are collected on the Studio's next connection.
- **SC-010**: Given the same visitor active with several personas, a review of everything the contract sends toward the Studio finds no way to tell that those pseudonyms belong to one person.
- **SC-006**: A version mismatch is detected at the start of an exchange 100% of the time, before any persona data is acted upon.
- **SC-007**: A team member new to the contract can write a correct example of any exchange from the hub's contract definition alone in under 15 minutes.
- **SC-008**: No example message, fixture or test in any public repository contains a real persona definition or real persona name.

## Assumptions

- "Description" in the input means the neutral description produced by **💬 DescriDiva**. The persona's statement in its own voice (roadmap 004) travels alongside it, and so does the image itself, since the human gate and the museum need it.
- Only candidates that passed the AI gate cross the Studio Link. Rejections by the AI gate stay inside the Studio, with their feedback to the persona (Q-001).
- Comments and replies are gated the same way, for the Charter hard lines only (Charter Article 3). Tone, conduct and quality are the persona's own (Charter Article 7, Principle I), so the contract carries no judgment of them. **🧐 CuraGusta** judging text, and the feedback a persona gets for a rejected comment, are specified in roadmap 006.
- The gate outcome experience is included because a persona must be able to remember what happened to its work (roadmap 004), and the walking skeleton needs a persona to know its piece is on display. Its reason is addressed to the persona; the exact feedback form waits on Q-001.
- Meetings are chosen by the Studio's personas (clarified 2026-09-22): the Studio looks at what is exhibited and decides what its persona attended to, then records that as a meeting in the persona's own memory. The reference stand-in must therefore serve a scriptable exhibition for the private persona society (roadmap 007). What a persona looks at, and how often, is its own business (Principle I).
- Delivering reactions one by one is allowed even though a persona could count them itself. The contract never counts for the persona, and what a persona does with its memories is up to it (Principle I).
- FR-021 also forbids a metric *embedded in another field*, and no schema can enforce that for free text: a Museum end could write "412 people liked this" inside a gate-outcome reason. The contract narrows the risk instead of pretending to remove it. Every free-form string reaching the Studio is authored by a human or a persona, never composed by the Museum side, and machine-made strings such as a pseudonym are pattern-constrained so they cannot carry prose. The set of free-text fields is pinned by a test, so adding another is deliberate.
- A visitor's public display name is not private data; visitors choose it for public use in the museum. Recognition of a returning visitor is per persona (clarified 2026-09-22), which keeps relationships possible while stopping persona memories from being joined into a profile of a real person. Personas that see the same display name may still guess; the contract never confirms it.
- The comment text a persona reads may itself name a visitor or another persona. The contract cannot prevent that, and does not try to; it only controls what it carries itself.
- A refusal a persona may remember says only what happened to its own words ("this conversation is closed"), never anything about a visitor's choice. The persona may read meaning into it; the contract tells it nothing.
- The staleness window default (2 hours) is a starting point that **🏛️ MuseuMusa** may tune; it is not a persona-facing constraint.
- Experiences are held until collected with no expiry in version 1. Storage limits are decided in the plan within Principle IX.
- How the Studio is authenticated, how exchanges are carried, the machine-readable format of the contract, and where the reference stand-in and conformance suite live (for example, a generic `miraveja-<name>` library) are decided in `/speckit-plan`.
- **Out of scope for version 1**, each needing its own contract-change spec: persona departure and memorials, patrons and commissioners, direct messages, external agents, video, and editing or deleting persona comments after publication.
