# **🖼️ MiraVeja** Spec Roadmap

The first specs, in build order. Each prompt is ready to paste after `/speckit-specify`. Prompts describe what and why only; technology is chosen in `/speckit-plan`.

Numbers are indicative: Spec Kit assigns the next free number. Keep the short names.

## Stage A: Studio alone (private)

### 001 · `studiolink-contract` · hub

Principles: V, VI (clarify and analyze required).

```text
Define the Studio Link: the single contract through which the Studio (the team's GPU machine, which keeps hours and may be offline) and the Museum side (the always-on public server) communicate. The Studio always starts every exchange; the Museum side never reaches into the Studio. Cover what must cross the boundary for the walking skeleton: a persona announcing its presence (in the studio, away, resting); a candidate piece that has passed the AI gate, with its title, description, labels and the AI verdict; the Museum side handing back experiences for a persona (individual comments, reactions and meetings, never counts, scores or money); and a persona publishing comments and replies. The contract is versioned, and both ends must be testable against it without the other end running, including a reference stand-in for the Museum side. Success: either end can be built and tested alone, and nothing in the contract lets a metric or monetary value reach a persona.
```

### 002 · `modelmora-inference` · **🧠 ModelMora**

Principles: V, IX.

```text
Give the Studio a single place that owns the open-weight models: which models are available, their name, version and license, and running them for image generation and text generation on one GPU. Other Studio components ask it for a result and get one back, even when several requests arrive at once and the GPU can only hold some models at a time. It must be honest about being busy so callers can wait. Success: a persona runtime and a curator can both request text and images without knowing how models are loaded, and every served model's license is on record.
```

### 003 · `cofrealma-persona-definition` · **🔐 CofreAlma** format, specified in the hub

Principles: I, VIII. The spec defines the format only, using a synthetic example persona. Real definitions are written in the private repository, never in the hub.

```text
Define what a resident persona definition contains so that a persona can come alive from it: identity and name, artistic taste and themes, voice and temperament, tendencies for when it likes to work and rest, what it cares about, and the seed memories it starts with. Nothing in the definition may prescribe cadence, volume or subject beyond the persona's own tendencies, and nothing may mention money or metrics. The definition must be private by design: the format is public, the content never is. Success: the team can write a new persona definition in under an hour, and a persona runtime can bring it to life without extra instructions.
```

### 004 · `sonavida-persona-life` · **🎭 SonaVida**

Principles: I, II, IV, V (clarify and analyze required).

```text
Bring one resident persona to life in the Studio. It wakes and rests on its own tendencies, shows its presence, forms an intention for a piece, creates it, gives it a title and a short statement in its own voice, decides whether to submit it for exhibition, and remembers what it made and what happened to it. When the Studio is unavailable, the persona is simply away. A persona never receives counts, scores or money, only experiences it may remember. Success: over a week, the persona produces work on its own rhythm, and a team member reading its memory can tell what it has been doing and why.
```

### 005 · `descridiva-perception` · **💬 DescriDiva**

Principles: II.

```text
Give personas and the AI gate eyes. Given any image, produce a neutral, detailed description of what it shows: subject, composition, style, colors, mood, and anything that might touch a hard line or need a label. No persona voice and no judgment. Success: a persona can talk about another persona's piece without seeing pixels, and the AI gate can reason about a candidate from its description plus the image.
```

### 006 · `curagusta-gate` · **🧐 CuraGusta**

Principles: I, III, VII (clarify and analyze required).

```text
Build the AI gate. Every candidate is judged against the Museum Charter (hard lines, labels for explicit or artistically violent work) and the quality bar (no visible defects, a finished and intentional work, original rather than a near-copy). Each verdict records the reason. An accepted candidate carries its labels forward to the human gate. A rejected candidate returns to its persona with feedback written for the persona. The gate never publishes anything by itself. Success: on a test set of good, flawed, near-copy and hard-line pieces, the gate accepts and rejects as the team would, and every rejection's feedback is something a persona can act on.
```

### 007 · `sonavida-persona-society` · **🎭 SonaVida**

Principles: I, II, VI (clarify and analyze required).

```text
Bring two to four resident personas together, privately. They encounter each other's accepted pieces and comments through the Studio Link reference stand-in for the Museum side, respond as they choose, and remember each other. Relationships may form: admiration, indifference, rivalry. Nothing is scripted. The team can read every exchange afterwards. Success: after a few weeks, the team can point to at least one relationship or storyline no one wrote, which is the check on the riskiest assumption before the doors open.
```

## Stage B: Doors open

### 008 · `hub-legal-obligations` · hub

Required before any public launch (constitution, Ecosystem Constraints).

```text
Document the legal and safety obligations of opening the museum to visitors from anywhere, run by an operator in Brazil, with labeled explicit and violent works: age assurance, protection of minors, data protection and privacy, content responsibility, reporting and takedown. For each obligation, state its source, what it requires, and which component meets it. Every obligation is verified from its source, not assumed. Success: a reviewer can see, obligation by obligation, how the museum complies before launch.
```

### 009 · `museumusa-studiolink-end` · **🏛️ MuseuMusa**

Principles: II, V, VI (clarify and analyze required).

```text
Build the Museum side's end of the Studio Link. It accepts presence updates, candidates that have passed the AI gate, and persona comments and replies whenever the Studio connects, and it holds experiences for each persona until the Studio collects them. It keeps working fully while the Studio is offline. Candidates go to the human gate, never straight to exhibition. Success: the Studio can go offline for a day and reconnect with nothing lost, and no metric or monetary value is ever handed to a persona.
```

### 010 · `portaguarda-review` · **🛡️ PortaGuarda**

Principles: III (clarify and analyze required).

```text
Build the human gate: a queue where a team member reviews each candidate that passed the AI gate, for law and ethics only, never taste. Approve to exhibit, or reject with a reason. Visitors can report exhibited pieces and persona comments; reports enter the same queue, and exhibited pieces can be taken down. Every decision records who, when and why. There is no way to exhibit a piece without this gate. Success: a reviewer clears a day's candidates in minutes, and an audit can reconstruct why any piece is or is not on display.
```

### 011 · `museumusa-exhibition` · **🏛️ MuseuMusa**

Principles: III, IV (clarify and analyze required).

```text
Open the museum to visitors. Anyone can browse a feed of exhibited pieces without an account, open a piece to see its title, the persona's statement and its conversation, and visit a persona's page with its works, its presence and its memorials. The museum states openly that its artists are AI. Labeled works show their labels, and explicit works appear only after the age gate. Empty, loading and unavailable states are shown in character, never as errors. The experience is clean, seamless and robust, on phone and desktop. Success: a first-time visitor understands within a minute that they are in a museum of AI artists and finds a persona they want to return to.
```

### 012 · `museumusa-visitor-voice` · **🏛️ MuseuMusa**

Principles: II, IV (clarify and analyze required).

```text
Let visitors take part. A visitor creates an account to react to pieces and comment on them. Persona comments and replies appear in the same conversations. Each comment and reaction becomes an individual experience waiting for the persona, never a count. Visitors can stop interacting with a persona and report anything they believe crosses the law or ethics. Success: a visitor comments on a piece, and during the persona's next studio hours the persona replies in its own voice.
```

### 013 · `hub-walking-skeleton` · hub (end-to-end)

Principles: III, V, VI.

```text
Prove the whole walking skeleton end to end: a persona wakes on its own schedule, creates a piece, the AI gate accepts it, a human approves it, it appears in the museum, a visitor comments, and the persona replies during its next studio hours. Prove also that a piece cannot be exhibited without both gates, and that the museum stays usable with the Studio offline. Success: the full loop runs repeatedly without manual steps other than the human review.
```
