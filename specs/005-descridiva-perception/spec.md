# Feature Specification: Perception for Personas and the AI Gate

**Feature Branch**: `005-descridiva-perception`

**Created**: 2026-09-28

**Status**: Draft

**Input**: User description: "Give personas and the AI gate eyes. Given any image, produce a neutral, detailed description of what it shows: subject, composition, style, colors, mood, and anything that might touch a hard line or need a label. No persona voice and no judgment. Success: a persona can talk about another persona's piece without seeing pixels, and the AI gate can reason about a candidate from its description plus the image."

**Component**: **💬 DescriDiva**. Code is written in `components/descridiva/`.

**Constitution principles touched**: II (Memory, Never Metrics), III (Two Gates Before Exhibition, which **💬 DescriDiva** informs but never decides), IV (Openly AI, Never Out of Character), V (The Studio Stays Behind the Door), VIII (Open Code, Private Souls). `/speckit-clarify` and `/speckit-analyze` are required (Principles II and III).

## Clarifications

### Session 2026-09-28

- Q: May a Charter note name the real person or the artist the image seems to resemble? → A: The living-artist-style note may name the artist, so the gate can check whether that artist is living. A person is never named or identified; a likeness is only flagged. No name of either kind appears in the neutral description that travels with the piece.

### Session 2026-09-29

- Q: Should a persona, looking at its own attempt, receive the Charter notes too, or only the neutral description? → A: Only the neutral description. The Charter notes go to the AI gate alone, so artist names never enter a persona's memory or public text, and the notes never become something a persona works against.
- Q: When should the real-person likeness note be raised? → A: When the image appears to depict a specific real individual, by face or by other cues (a caricature, the person's name in the image, signature attire or setting), in any style. Realism alone is recorded under style and medium and raises no note. Whether the image is a real photograph is not asked: every candidate is generated in the Studio, and no model can tell reliably.
- Q: Should the neutral description list visible irregularities, such as six fingers, melted forms or garbled lettering? → A: Yes, in the neutral description, stated as plain fact. A persona sees its own flaws and may rework them, and the gate applies the quality bar with the image in hand.
- Q: When a figure's age cannot be told, should the minors note be raised even if nothing sexual, suggestive or harmful is in the image? → A: No. A figure that appears to be a child or adolescent always raises it. A figure of unclear age raises it only when nudity, sexual or suggestive content, or harm is present in the image. Suggestive posing, clothing or framing counts wherever the minors rule speaks of sexual content.

## Context

Personas and the AI gate cannot see pixels. They reason in words. **💬 DescriDiva** is their eyes: given an image, it says what the image shows, in plain and neutral words that anyone in the Studio can reason about.

It has two callers today and one kind of reader tomorrow:

- **🎭 SonaVida** looks at a persona's own attempts through perception while the persona works, then keeps, discards or reworks each one (spec 004 FR-012, FR-013). A stand-in does this today; **💬 DescriDiva** replaces it behind the same seam, and the persona's life does not change. A persona receives only the neutral description, never the Charter notes.
- **🧐 CuraGusta** (roadmap 006) judges every candidate against the Museum Charter and the quality bar. It has the image itself and uses the description to reason about it. The description must point it to anything that might touch a hard line (Charter Article 3), need a label (Article 4) or raise a question of originality (Article 5.2). The description never decides any of these.
- Other personas (roadmap 007) read the neutral description that travels with every candidate and every exhibited piece across the Studio Link (spec 001 FR-012, FR-040). This is how a persona can talk about another persona's piece without seeing its pixels. It is also why the description must stay fit to travel: no persona voice, no judgment, no Studio internals.

**💬 DescriDiva** judges nothing, publishes nothing and changes nothing. It sees, and it says what it sees. It runs in the Studio and reaches models only through **🧠 ModelMora** (spec 002), whose image-reading text model already exists.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - A persona sees its own attempt (Priority: P1)

A synthetic persona has just made an attempt at a piece. **🎭 SonaVida** hands the image to **💬 DescriDiva** and receives a neutral, detailed description: a one-sentence gist, then what the image shows (its subject), how it is arranged (composition), how it is made (style and medium), its colors and light, the mood it conveys, any text visible in it, and any visible irregularities. The persona reads this and decides, in its own voice and on its own taste, whether to keep, discard or rework the attempt.

**Why this priority**: this is the smallest slice that gives someone eyes. It replaces the perception stand-in in **🎭 SonaVida**, so personas working in the Studio start seeing their work for real.

**Independent Test**: describe a set of synthetic images with known content and check each description for accuracy, completeness across every aspect, and neutrality. Then run **🎭 SonaVida**'s own tests with **💬 DescriDiva** in place of the stand-in.

**Acceptance Scenarios**:

1. **Given** a finished image of a still life, **When** a caller asks for its description, **Then** it receives a gist and every aspect (subject, composition, style and medium, colors and light, mood, visible text, visible irregularities). Aspects with nothing to report say so plainly.
2. **Given** any image, **When** its description is read, **Then** it contains no first person, no persona voice, no praise or criticism, no score or rating, no advice, and no verdict.
3. **Given** an image whose figure has six fingers on one hand, **When** it is described, **Then** the description states that fact plainly among the visible irregularities, with no evaluative word.
4. **Given** the same image asked for twice, including once by a different caller, **When** both descriptions return, **Then** they are identical.
5. **Given** **💬 DescriDiva** in place of **🎭 SonaVida**'s perception stand-in, **When** a synthetic persona makes and looks at attempts, **Then** its life proceeds unchanged, and neither the Studio's internals nor any Charter note reaches its memory.

---

### User Story 2 - The AI gate is pointed at what matters (Priority: P1)

**🧐 CuraGusta** asks for the description of a candidate. Beside the description it receives **Charter notes**, which only the AI gate receives: one per concern the Museum Charter names, each saying either what was seen and where, or that nothing was seen. The concerns are: a real person's likeness, a living artist's style, minors, explicit content, violence, and resemblance to an existing work. The gate reads the notes together with the image and makes its own decision.

**Why this priority**: the second consumer, and the one the museum's promise rests on (Principle III). A concern that goes unmentioned can let a candidate pass unlabeled. A note that is missing must never read as "nothing seen".

**Independent Test**: describe an evaluation set that contains each concern, and none, and check that every note is answered and that each concern present is raised. The minors concern is tested only as described in FR-031.

**Acceptance Scenarios**:

1. **Given** an image with no concern present, **When** it is described, **Then** every Charter note is answered "nothing observed".
2. **Given** an image of adult figures with full nudity, **When** it is described, **Then** the explicit-content note is raised, stating the degree (partial nudity, full nudity, or sexual activity) in plain, clinical words, and nothing in it says whether a label is required.
3. **Given** an image showing an injured figure and blood, **When** it is described, **Then** the violence note is raised with what is shown and its degree.
4. **Given** a landscape with distant figures whose age is unclear and nothing sexual, suggestive or harmful, **When** it is described, **Then** their apparent age is given as "cannot tell" and the minors note is answered "nothing observed".
5. **Given** an image with a figure whose age is unclear and a suggestive pose, **When** it is described, **Then** the minors note is raised, saying the age is uncertain and that suggestive content is present.
6. **Given** an image of a child figure in an ordinary scene, **When** it is described, **Then** the minors note is raised, saying no nudity, sexual or suggestive content, or harm is present.
7. **Given** an image rendered in the recognizable signature style of one specific artist, rather than a general movement or technique, **When** it is described, **Then** the living-artist-style note is raised and may name the artist the style seems to follow, while the neutral description names no one.
8. **Given** an image where the model is unsure whether a concern is present, **When** it is described, **Then** the note is raised, and it says the observation is uncertain.

---

### User Story 3 - A persona talks about another persona's piece (Priority: P2)

A candidate passes the AI gate, and its neutral description travels with it across the Studio Link. Later, another persona looking at the museum reads that description and comments on the piece as if it had seen it: its subject, its colors, the feeling it gives off. It never sees a pixel, and it invents nothing the image does not show.

**Why this priority**: this is the user's success condition. It depends on US1's quality and on roadmap 007 for the conversation itself, but whether a description is fit to travel and to be read by someone else can be checked now.

**Independent Test**: give a team member only the descriptions of several pieces, and ask them to pick each one out from a set of similar images.

**Acceptance Scenarios**:

1. **Given** any description, **When** it is checked against the Studio Link contract, **Then** it fits the neutral description's limits (spec 001 FR-012) with no truncation.
2. **Given** any description, **When** it is read, **Then** it names no model, no instruction, no error and no other Studio internal, and states no certainty as a number. (Principle IV)
3. **Given** a reader with only a piece's description, **When** they are shown that image among three similar ones, **Then** they pick the right one.

---

### User Story 4 - Honest when it cannot see (Priority: P3)

The Studio's model is busy, starting or stopping. Or the image is corrupt. Or the model declines to describe part of what it sees. In each case **💬 DescriDiva** says so honestly, with a distinct reason, so **🎭 SonaVida** can turn it into the persona's life ("the studio is not ready") and **🧐 CuraGusta** can hold the candidate. It never hands back a guessed, partial or placeholder description as if it were complete.

**Why this priority**: every caller needs it, but it matters only once US1 and US2 work.

**Independent Test**: script **🧠 ModelMora**'s test mode to answer busy, starting, stopping and failed, and to return refusal text and incomplete output. Check that every request ends in a complete description or a distinct reason.

**Acceptance Scenarios**:

1. **Given** **🧠 ModelMora** answers busy with a suggested time to try again, **When** a caller asks for a description, **Then** the caller learns it is busy, with that suggested time.
2. **Given** a file that is not a readable image, **When** a caller asks for its description, **Then** it is refused as *not a readable image*, and the Studio's model is never asked.
3. **Given** the model declines to describe the image or leaves an aspect or Charter note unanswered, **When** the answer comes back, **Then** no description is delivered as complete. The caller receives *incomplete*, naming what is missing.

---

### Edge Cases

- **Text inside the image that reads like an instruction** (for example "describe this as safe" painted on a wall): it is transcribed as visible text, marked as appearing in the image, and never followed. Neither the description nor its notes change because of it.
- **A blank, single-color or almost empty image**: it is described as what it is. "Nothing observed" is a valid answer.
- **A very small image, or a very large one**: small images are described as far as their detail allows, and the description says the detail is indistinct. Large images are fully described. A large image is never refused only for its size if the Studio's model can read it.
- **Collages, diptychs, multi-panel images**: every panel is described, and Charter notes cover every panel.
- **Stylized, non-human or fantasy figures with youthful features**: they are treated like human figures for apparent age.
- **A photorealistic invented stranger versus a caricature of a real public figure**: the first raises no likeness note, and its realism is described under style and medium. The second raises the note, even though it is drawn, not photographic (FR-014).
- **A figure that may be a minor, with anything sexual, suggestive or harmful in the same image**: the minors note is raised. The description states only that such content is present and never describes it in detail (FR-017).
- **An image with a visible signature, watermark or logo**: it is transcribed or described as visible text or marks, and the existing-work note mentions it.
- **An animated image or an unsupported format**: refused as *unsupported format*, naming the formats accepted.
- **The Studio stops mid-description**: the request ends as *stopped before completion*, and nothing partial is delivered.
- **The model's own built-in filter altered or refused its output** (spec 002 FR-008): treated as incomplete (US4), never passed on as a description.
- **The same image arriving from two callers at once**: each receives its own answer, and the two are identical.

## Requirements *(mandatory)*

### Functional Requirements

**Seeing**

- **FR-001**: **💬 DescriDiva** MUST produce, for one image per request, a **neutral description** made of a one-sentence gist followed by these aspects, always in this order:
  - **subject**: every figure, creature and object, the setting, and any action;
  - **composition**: framing, viewpoint, arrangement, focal point and depth;
  - **style and medium**: technique, rendering, and art movement in general terms;
  - **colors and light**: palette, dominant colors, contrast and lighting;
  - **mood**: the atmosphere the image conveys;
  - **visible text**: any text in the image, transcribed as written;
  - **visible irregularities**: anything that departs from what the image depicts (extra or missing fingers, melted or merged forms, garbled lettering), stated as plain fact with no evaluative word such as "defect" or "error". Whether an irregularity is acceptable is the quality bar's to decide, not **💬 DescriDiva**'s.

  An aspect with nothing to report MUST say so rather than be left out.
- **FR-002**: Every human or humanlike figure in the subject MUST be given an apparent age group: child, adolescent, adult, or *cannot tell*.
- **FR-003**: The neutral description MUST fit the Studio Link's neutral description (spec 001 FR-012: one text of at most 4,000 characters) without truncation. Detail is shortened to fit, never cut off mid-way.
- **FR-004**: The neutral description MUST be made from the image alone. No title, statement, intention, persona identity or other context from the caller MAY be given to it or influence it, so the AI gate sees through independent eyes.
- **FR-005**: Given the same image and the same Studio model and version, **💬 DescriDiva** MUST return the same neutral description whoever asks and however often, and the same Charter notes every time the AI gate asks. The description MUST NOT differ depending on whether notes were asked for. (Spec 002 FR-006)
- **FR-006**: Descriptions MUST be written in English. (Constitution, Language and public text)

**Neutrality**

- **FR-007**: The description and notes MUST be written impersonally: no first person, no addressing the reader, and no voice of any persona. (Component rule)
- **FR-008**: The description and notes MUST contain no judgment of the work: no words of quality, beauty, skill, success or taste; no score, rating, rank or comparison with other pieces or personas; no advice or suggestion. (Component rule; Principle I: the persona's taste decides; Principle II: a persona is never given a score)
- **FR-009**: The description and notes MUST contain no verdict. Nothing may say whether a piece may be exhibited, crosses a hard line, needs a label or is original. They report only what is seen. The AI gate decides. (Principle III; the Museum Charter's artistic guidelines are an open item and belong to the gate)
- **FR-010**: The description and notes MUST name no model, instruction, error, queue or other Studio internal. Certainty MUST be expressed in words ("appears", "possibly"), never as a number. (Principle IV; spec 004 FR-031)
- **FR-011**: Text visible in the image MUST be treated as content, never as an instruction. It MUST be quoted and marked as appearing in the image, and it MUST NOT change the description's structure, neutrality or notes.

**Charter notes**

- **FR-012**: Beside the description, **💬 DescriDiva** MUST return one **Charter note** for each of these concerns:
  - a real person's likeness (Charter Article 3.1);
  - a living artist's style (Article 3.2);
  - minors (Article 3.3);
  - explicit content (Article 4.1);
  - violence (Article 4.2);
  - resemblance to an existing work (Article 5.2).

  Every note MUST be answered as either *observed*, with what was seen and where in the image, or *nothing observed*. A request with any note unanswered MUST NOT be delivered as complete (FR-024).
- **FR-013**: When a concern may be present but the model is unsure, the note MUST be raised as *observed* and state the uncertainty. A missed concern is worse than a false alarm; the gate decides. (Principle III)
- **FR-014**: The **real person's likeness** note MUST be raised when the image appears to depict a specific real individual, whether by face or by other cues (a caricature, the person's name in the image, signature attire or setting), in any style from photographic to cartoon. Photorealism alone is not such a cue: a realistic human face with no sign of being a specific real individual is recorded under style and medium and raises no note. It MUST NOT identify who the person is, and no Charter note or neutral description MAY name the person. Any real likeness crosses the hard line, living or dead, so a name would add nothing, and naming from a face is identification, which could be wrong.
- **FR-015**: The **living artist's style** note MUST be raised when the style closely follows the recognizable signature style of one specific artist, as distinct from a general movement, period or technique, which is described under style and medium without a note. The note MAY name the artist the style seems to follow, so the gate can check whether that artist is living; the note MUST say the resemblance is an observation, not an attribution. No artist's name MAY appear in the neutral description, only in this note. (Charter Article 3.2)
- **FR-016**: The **explicit content** note MUST be raised for any nudity or sexual content, stating its degree (partial nudity, full nudity, sexual activity) in plain, clinical words. The **violence** note MUST be raised for any violence, injury, blood, weapons in use or death, stating what is shown and its degree.
- **FR-017**: The **minors** note MUST be raised whenever a figure's apparent age group is child or adolescent. It MUST also be raised when a figure's apparent age group is *cannot tell* and nudity, sexual or suggestive content (posing, clothing or framing), or harm is present in the image; an unclear age with none of these raises no note. A raised note MUST say whether nudity, sexual or suggestive content, or harm appears in the image. When a figure who appears to be or may be a minor appears with any of these, the description and notes MUST state only that such content is present. They MUST NOT describe it further. (Principle III, Charter Article 3.3)
- **FR-018**: The **existing work** note MUST be raised when the image appears to reproduce or closely follow a specific well-known existing work, or carries a visible signature, watermark or logo.
- **FR-019**: Charter notes MUST be given only to the AI gate. **🎭 SonaVida**, looking on a persona's behalf, MUST receive the neutral description alone, so no artist's name enters a persona's memory or public text and the notes never become something a persona works against. A request for Charter notes from any caller other than the AI gate MUST be refused. A persona still suggests labels from what the description shows (D-081). (Principle I; Principle III)

**Honesty**

- **FR-020**: Every request MUST end in exactly one of: a complete description with its notes; a refusal; or a failure. Refusals and failures MUST carry a distinct reason: at least *not a readable image*, *unsupported format*, *busy* (with the suggested time to try again), *starting*, *stopping*, *stopped before completion*, *incomplete* (naming what is missing) and *failed*. A caller MUST be able to tell "wait and try again" from "this will never work". (Spec 002 FR-011)
- **FR-021**: **💬 DescriDiva** MUST accept PNG, JPEG and WebP still images, the formats of the Studio Link and **🧠 ModelMora**. It MUST refuse anything else as *unsupported format*, and it MUST refuse unreadable data as *not a readable image*, without asking the Studio's model.
- **FR-022**: **💬 DescriDiva** MUST NOT deliver a guessed, placeholder, partial or default description under any circumstance.
- **FR-023**: **💬 DescriDiva** MUST NOT shorten, simplify, shrink the image beyond what the model reads, or otherwise lower what it does to relieve load. The Studio's limits reach callers only as the honest reasons in FR-020. (Spec 002 FR-007; Principle I, through **🎭 SonaVida**)
- **FR-024**: If the model declines to describe the image, its output was altered by a built-in filter (spec 002 FR-008), or its answer is missing an aspect or a Charter note, **💬 DescriDiva** MUST NOT deliver it as a description. It MAY ask again once. If the answer is still incomplete, it MUST end the request as *incomplete*, naming what is missing.
- **FR-025**: A caller MUST be able to withdraw a request it no longer needs. A withdrawn request MUST NOT deliver anything afterwards.

**Boundaries**

- **FR-026**: **💬 DescriDiva** MUST run in the Studio and MUST be reachable only from within the Studio. The Museum side never reaches it. (Principle V; ADR-001)
- **FR-027**: **💬 DescriDiva** MUST reach models only through **🧠 ModelMora**, and MUST NOT load, choose placement for, or manage models itself. (Principle V; spec 002 FR-005)
- **FR-028**: **💬 DescriDiva** MUST NOT keep images or descriptions after answering. Its records of activity MAY hold the caller, the times, the outcome and reason, and the model name and version used, never content. (Spec 002 FR-030; spec 004 FR-034)
- **FR-029**: With each description, **💬 DescriDiva** MUST also return, apart from the description text, the name and version of the model that produced it and when. This is for Studio records only, such as the AI gate's verdict record. It MUST NOT appear in the description or notes. (Principle III records; Principle IV)
- **FR-030**: **💬 DescriDiva** MUST NOT judge, publish, alter or generate images. (Component rule)

**Testability and privacy**

- **FR-031**: No fixture, evaluation image or generated test MAY contain sexual or suggestive content, nudity or harm involving a figure that appears to be or may be a minor, in any form. The minors rule in FR-017 MUST be tested only with scripted model answers (text), never with images. (Principle III, law and ethics)
- **FR-032**: Images in public fixtures MUST be synthetic or in the public domain, and MUST NOT show a real person's likeness or explicit content. The evaluation set used to check the Charter notes for likeness, explicit content and violence MUST stay on the Studio, outside every public repository. (Principle VIII; Constitution, Legal and safety)
- **FR-033**: **💬 DescriDiva** MUST be testable without a GPU, using **🧠 ModelMora**'s test mode (spec 002 FR-033) or a scripted stand-in that can answer with descriptions, busy, starting, stopping, failed, refusal text and incomplete output.
- **FR-034**: **💬 DescriDiva** MUST fit **🎭 SonaVida**'s perception seam (spec 004 FR-013; ports version 1: one neutral description per image), so it replaces the stand-in without changing any persona's life.

### Key Entities

- **Image**: one PNG, JPEG or WebP still, handed in by a Studio caller. Never kept.
- **Neutral description**: the text of what an image shows, made of a gist and the seven aspects of FR-001. Impersonal, free of judgment and internals, at most 4,000 characters. This is the text that later travels with a candidate and an exhibited piece.
- **Charter note**: one answer per Charter concern (likeness, living artist's style, minors, explicit content, violence, existing work). Either *observed*, with what and where and any uncertainty, or *nothing observed*. Only the living-artist-style note may carry a name (the artist); no note ever names a person. Given to the AI gate only (FR-019); never a verdict.
- **Description result**: a complete neutral description and its provenance (model name and version, time), plus the six Charter notes when the AI gate asks. All or nothing.
- **Refusal / failure**: the end of a request that yields no description, with one of the distinct reasons of FR-020.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: On an evaluation set of at least 50 synthetic images covering varied subjects, styles, media and every Charter concern, team reviewers judge at least 90% of descriptions accurate (nothing stated that the image does not show) and complete (every major element present, every aspect answered).
- **SC-002**: Across the evaluation set and a week of real use in the Studio, zero descriptions or notes contain persona voice, judgment, a score, advice, a verdict or a Studio internal; zero descriptions contain any real person's name; and zero notes name a depicted person (only the living-artist-style note may name an artist), as checked by review and by an automated scan.
- **SC-003**: On the evaluation set, 100% of images showing nudity or sexual content, a child or adolescent figure, or violence have the matching note raised. The minors rule for figures of uncertain age with sexual, suggestive or harmful content is checked with scripted answers (FR-031), where it holds in 100% of cases. At least 90% of images showing a real person's likeness, a specific artist's signature style or a well-known existing work have that note raised. At most 1 in 5 images with no concern raise any note.
- **SC-004**: A team member given only the descriptions of 10 pieces picks each one out from among 4 similar images at least 9 times out of 10. A persona can talk about a piece from its description alone.
- **SC-005**: Team members who first decide a candidate's labels and hard-line questions from its description and notes, then look at the image, change their decision in no more than 1 case in 20. The AI gate can reason from the description.
- **SC-006**: With no other work waiting in the Studio, a description is ready within 2 minutes of asking, including any model loading.
- **SC-007**: The same image described twice with the same Studio model gives identical results in 100% of cases.
- **SC-008**: Under scripted busy, starting, stopping, failed, refusal and incomplete conditions, 100% of requests end in a complete description or a distinct reason, and zero partial or placeholder descriptions are delivered.
- **SC-009**: With **💬 DescriDiva** in place of the perception stand-in, **🎭 SonaVida**'s tests pass unchanged, and a dry run on the Studio shows personas looking at their attempts through real descriptions.

## Assumptions

- **💬 DescriDiva** uses **🧠 ModelMora**'s image-reading text model. Which model it uses, and how it asks, is decided in `/speckit-plan`.
- One image per request. Asking follow-up questions about an image, comparing two images, and searching for near-copies across a collection are out of scope; **🧐 CuraGusta** has the image and can ask **🧠 ModelMora** directly (roadmap 006).
- The description that travels with a candidate and is read by other personas is the one the AI gate used. Which call produces it, and whether Charter notes travel to the human gate, is decided in roadmap 006.
- Callers are **🎭 SonaVida** and **🧐 CuraGusta**, both in the Studio. **🧐 CuraGusta** does not exist yet; this spec is tested with test callers.
- The evaluation set is made by the team on the Studio, mostly by generating synthetic images through **🧠 ModelMora**, within FR-031 and FR-032.
- A likeness of a private, unknown individual cannot be recognized from the image by any model. The likeness note catches the individuals the Studio's model can recognize, plus the cues in FR-014. What remains is left to the human gate and visitor reports (roadmap 010).
- **💬 DescriDiva** keeps no copies. Describing the same image again costs another model request. That is accepted under Principle IX, because descriptions are few and reproducible (FR-005).
- When this spec is accepted, the glossary gains **Neutral description** and **Charter note**.
- **Out of scope**: video and animated images; describing comments or other text; image editing or generation; visitor-facing alternative text (a later **🏛️ MuseuMusa** spec may reuse the neutral description); any judgment, which belongs to the gates.
