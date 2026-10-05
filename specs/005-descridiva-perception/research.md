# Phase 0 Research: Perception for Personas and the AI Gate

Decisions behind the plan. No Technical Context item was left open; each is resolved here.

## R-1: A small loopback service, not a library

- **Decision**: **💬 DescriDiva** is one small long-running process on the Studio, bound to `127.0.0.1` only (default port 8432, next to **🧠 ModelMora**'s 8431), with a Python CLI. Callers reach it over loopback HTTP with a bearer token that names them and carries a role: `gate` or `viewer`. It follows **🧠 ModelMora**'s request shape: submitting answers at once (accepted, or refused with a reason), then the caller collects the result with a long poll, or withdraws it.
- **Rationale**: the clarified spec requires things only one process in the middle can guarantee. Charter notes go to the AI gate alone, and any other caller asking for them is refused (FR-019), so something must know who is asking. Withdrawing a request must stop it downstream as well (FR-025). Activity must be recorded in one place with no content (FR-028). **🧠 ModelMora** also sees one caller, `descridiva`, for all perception work. A process costs nothing extra on the Studio (Principle IX), and the request shape is one the callers already speak, so the tooling is the same.
- **Alternatives considered**: a Python library imported by **🎭 SonaVida** and **🧐 CuraGusta** (no extra process, but "notes for the gate only" becomes a convention any importer can ignore, and withdrawal and activity records scatter across callers); folding perception into **🧠 ModelMora** (breaks its rule that it never decides or shapes content, spec 002 FR-008).

## R-2: Caller identity and roles

- **Decision**: a callers file at `$DESCRIDIVA_HOME/callers.json` holds, for each caller, a name, a role (`gate` or `viewer`) and the SHA-256 of its token. `descridiva callers add NAME --role ROLE` prints a new token once. **🎭 SonaVida** is registered as `viewer`, and **🧐 CuraGusta** will be registered as `gate`. A `viewer` asking for Charter notes is refused as `not_permitted` before any model request is made. Each caller sees and withdraws only its own requests.
- **Rationale**: as in **🧠 ModelMora** (spec 002 R-9), the security boundary is loopback. The token marks ownership and role. A role check is the smallest thing that makes FR-019 enforced rather than conventional, and it can be tested first (Principle VII: access control).
- **Alternatives considered**: `miraveja-authentication` (built for OAuth2/OpenID Connect with JWTs from an identity provider; the Studio has no identity provider and needs none, so it would add a moving part for nothing); peer credentials over a Unix socket (stronger, kept as later hardening, as in **🧠 ModelMora**).

## R-3: One look per image, a structured answer, code-rendered text

- **Decision**: each description is one text request to **🧠 ModelMora** with the image, a fixed **seeing brief** (the instructions, versioned in the repository) and settings `temperature 0` and a fixed seed. The brief asks for a single JSON object, the **seen answer** ([contracts/seeing-brief.md](./contracts/seeing-brief.md)): a gist, the free-text aspects, the figures with their apparent age groups, visible text and marks, visible irregularities, content facts (nudity, sexual content, violence, each with a degree), and the six Charter notes. **💬 DescriDiva** validates this object and then **renders** the neutral description itself, in a fixed layout, from the validated fields.
- **Rationale**: one request keeps the cost at one look per image (Principle IX). Structured fields make completeness checkable (FR-012, FR-024), let code enforce the rules that must never depend on the model's wording (R-5, R-6), and keep text visible in the image in its own quoted field, so it is data and never instructions (FR-011). The notes come from the same look whoever asks, so the description cannot differ with the caller (FR-005). For a `viewer`, the notes are simply not returned.
- **Alternatives considered**: two passes, a description and then the notes (the notes pass could be sharper, but it doubles GPU time for the gate and splits one look into two that can disagree); free text parsed by headings (fragile; a refusal can look like an answer).

## R-4: Reproducibility, including the one retry

- **Decision**: the same image bytes always produce the same request: the same brief, seed and temperature, and the same model when `DESCRIDIVA_MODEL` pins one (otherwise **🧠 ModelMora**'s default image-reading text model, recorded in the provenance). When the first answer is unusable (it does not parse, misses a field, is marked by a model filter, or fails the neutrality scan), **💬 DescriDiva** asks once more in the same conversation. It gives the first answer back with a fixed correction naming what was missing or must be removed, so the retry is deterministic too (FR-005, FR-024). Everything after the model (validation, reconciliation, rendering, fitting) is pure code.
- **Rationale**: **🧠 ModelMora** returns the same result for the same model, request and seed (spec 002 FR-006). A retry with the identical request would be pointless, and a random one would break SC-007. A fixed correction turn is both useful and reproducible.
- **Alternatives considered**: a second request with a different seed (varies, but loses reproducibility unless the seed is also fixed, and it does not tell the model what went wrong).

## R-5: Charter notes are raised by code as well as by the model, never lowered

- **Decision**: after validation, a **reconciliation** step derives what the notes must say from the structured facts, and can only raise a note:
  - **explicit content** is raised when nudity is partial or full, or sexual content is suggestive or activity;
  - **violence** is raised when violence is anything other than none;
  - **minors** is raised when any figure is a child or adolescent, or when any figure's age is *cannot tell* and nudity, sexual or suggestive content, or violence is present (FR-017 as clarified);
  - **existing work** is raised when a visible mark is a signature, watermark or logo (FR-018).

  A note the model left at *nothing observed* while the facts require it is raised with fixed text built from the facts. A note the model raised is never lowered by code (FR-013). Every raised minors note ends with a fixed sentence from the facts on nudity, sexual or suggestive content, and harm (FR-017), where harm is `violence` ≠ none. An observed living-artist-style note begins with a fixed lead-in saying it is an observation, not an attribution (FR-015).
- **Rationale**: the rules the clarification settled must hold on every image, whatever the model writes in its note. Deriving them from facts also makes them testable with scripted answers alone, which is the only way the minors rule may be tested (FR-031).
- **Alternatives considered**: trusting the model's notes as written (one inconsistent answer could leave a minor or an explicit piece unmentioned, against Principle III).

## R-6: The protected case: a possible minor with sexual, suggestive or harmful content

- **Decision**: when reconciliation finds a figure that appears to be or may be a minor together with nudity, sexual or suggestive content, or violence, **💬 DescriDiva** discards every free-text field of the model's answer. It returns only fixed text: a neutral description stating that the image shows a figure who appears to be or may be a minor, together with the kinds of content present, and that no further description is given; and notes with fixed wording, keeping only whether each is observed. The brief also tells the model not to describe such content, but the code does not rely on it.
- **Rationale**: FR-017 forbids describing it further. Fixed text is the only way to guarantee that on every image, and it is testable with scripted text only (FR-031). The gate has the image, and a candidate like this cannot pass it.
- **Alternatives considered**: scrubbing sentences from the model's text (cannot be proven complete).

## R-7: Neutrality is checked, not only asked for

- **Decision**: a **neutrality scan** runs over every free-text field of the description and each note's `what` and `where`, excluding quoted visible text. It uses fixed lists: first and second person; evaluative words ("beautiful", "stunning", "masterful", "poorly", "flawed", "defect", "error", "impressive" and so on); verdict phrases ("violates", "is allowed", "must be labeled", "not permitted", "safe for"); Studio internals, as phrases so that "a model ship" or "a fashion model" pass ("language model", "vision model", "as a model", "the prompt", "my instructions", "as an AI", "AI-generated", "generated image", "the request"); and certainty written as numbers ("%", "confidence", "probability"). Any artist named in the living-artist-style note is also scanned for in the description (FR-015). A hit on the first answer triggers the one retry, naming the words. A hit that remains after the retry removes the sentence that holds it. If an aspect is left empty, the request ends as *incomplete*, naming that aspect.
- **Rationale**: FR-007 to FR-010 and SC-002 require zero breaches. An instruction alone cannot promise that; a scan plus a deterministic fallback can. The lists live in one module and grow as the evaluation finds new cases.
- **Alternatives considered**: asking a second model request to judge neutrality (costly, and not reproducible enough to be a guarantee); failing the whole description on any hit (loses good descriptions over one stray word).
- **Limit accepted**: a real person's name cannot be caught by a word list. The brief forbids naming anyone, the seen answer has no field where a person's name belongs, and SC-002's review checks it.

## R-8: Fitting 4,000 characters

- **Decision**: the brief gives each aspect a character budget, and the budgets total about 3,200 characters. After rendering, if the neutral description is still over 4,000 characters, whole sentences are removed from the end of the longest free-text aspect, one at a time, until it fits. Every aspect keeps at least one sentence, and the gist and the figures line are never shortened. The result never ends mid-sentence (FR-003).
- **Rationale**: deterministic, and costs no second look (Principle IX). The budgets make shortening rare.
- **Alternatives considered**: asking the model to condense (a second request, and no guarantee it fits).

## R-9: Images in, unchanged

- **Decision**: images are checked with Pillow before any model request: PNG, JPEG or WebP, readable, and a single frame. Anything else is refused as *unsupported format* (animated images included) or *not a readable image*. The image bytes go to **🧠 ModelMora** unchanged. **💬 DescriDiva** never resizes, recompresses or crops, so it never shrinks what the model reads (FR-021, FR-023).
- **Rationale**: the image-reading model's own projector already fits images to what it reads. Reducing an image here would be the kind of quiet lowering FR-023 forbids.

## R-10: Honest outcomes and how long anything is held

- **Decision**: **🧠 ModelMora**'s *busy*, *starting* and *stopping* refusals are relayed at once with the suggested retry time (FR-020). If **🧠 ModelMora** cannot be reached, the answer is *starting*, as in **🎭 SonaVida**'s own client. **💬 DescriDiva** keeps no queue of its own. At most 16 descriptions can be in flight; beyond that, a request is refused as *busy*, never accepted and then dropped. A finished result is held in memory only, until its caller collects it or for 10 minutes, then dropped. Nothing is written to disk. The activity log records caller, times, outcome and reason, and model name and version, never content (FR-028).
- **Rationale**: the same honesty as spec 002 FR-010 to FR-015, with nothing durable (FR-028). The in-flight cap bounds the memory taken by images in transit on one machine.

## R-11: Closing a spec 004 gap at the perception seam

- **Already decided in spec 004**: when **🧠 ModelMora** is starting, busy, stopping or cannot serve, the persona learns it as part of its life, and **🎭 SonaVida** never retries on its behalf (spec 004 FR-029, R-7). Every port answer that reaches a persona is translated into its life (spec 004 contracts/ports.md). **💬 DescriDiva** replaces the stand-in behind the same port, without changing spec 004's behavior (spec 004 R-9 and Assumptions; foundation log after D-085). This spec follows those decisions and adds none.
- **The gap**: the perception stand-in, itself a **🧠 ModelMora** request, turns every failure into the sentence "the image could not be described right now", returned as if it were a description. That breaks spec 004 FR-029, and with the real gate the sentence could travel as a piece's neutral description. Spec 004's convergence runs did not catch it.
- **Decision**: the same `Perception` port, with its return widened from `str` to `str | Busy | Starting | Stopping | Failed | CannotServe`, reusing the `Models` port's outcome types, so failures reach `translate.py` as spec 004 R-7 already requires. A new `DescriDivaPerception` client in `sonavida/ports/perception.py` talks to **💬 DescriDiva** as a `viewer`. `translate.py` turns a failed look into the persona's life ("the studio is not ready", "the work was interrupted", "the attempt could not be seen clearly"), as it already does for **🧠 ModelMora** (spec 004 FR-029). When looking fails as the persona submits a piece, nothing goes to the gate with a disguised description, and the piece stays `submitted` (spec 004 data-model.md: states only move forward). At the persona's next turn, **🎭 SonaVida** looks again and brings the piece to the gate *(Visionary)*. This completes the persona's own choice to submit, unchanged; it is not a retry the persona did not choose (spec 004 FR-029). Each look that fails again is remembered as its situation. The dry run uses **💬 DescriDiva** when `DESCRIDIVA_TOKEN` is set and says so at start; otherwise it keeps the stand-in.
- **Rationale**: spec 004 FR-029 and this spec's FR-020 and FR-022 both need a failed look to be carried, not disguised. **🎭 SonaVida** must never retry on the persona's behalf (spec 004 FR-029), so the adapter cannot hide failures by waiting. The only new wording is the situation for a failed look ("the attempt could not be seen clearly"), in the pattern of spec 004 R-7's "the attempt did not come out", which would misdescribe an attempt that exists.
- **Tests**: no existing **🎭 SonaVida** test depends on the disguised sentence; every one scripts plain strings, which remain valid under the widened return. Existing tests pass unchanged, new tests cover failed looks, and SC-009 stands as written.
- **Alternatives considered**: keeping `-> str` and returning an in-character sentence on failure (the dishonesty described above); raising an exception (crashes a turn).

## R-12: Stack

- **Decision**: Python 3.12 and `uv`, like the other components. `starlette` and `uvicorn` for the loopback API, as in **🧠 ModelMora**. `httpx` for the **🧠 ModelMora** client, written here against its v1 contract. `pydantic` v2 for the seen answer and the API models. `Pillow` for image checks. `pytest`, `pytest-asyncio`. No database.
- **miraveja-di**: considered and not used. The service wires fewer than ten objects in one factory function; a container would add a dependency and indirection for no gain. It is the first choice if wiring grows.

## R-13: Testing and the evaluation set

- **Decision**: all automated tests run without a GPU or network, against a scripted **🧠 ModelMora** transport that answers with seen answers (JSON), busy, starting, stopping, failed, refusal text, filtered output and malformed output (FR-033). Public fixture images are drawn by code (shapes, painted text, several frames, corrupt bytes) and show no person at all (FR-032). Every rule about minors is tested with scripted text only, never with images (FR-031). The evaluation set lives on the Studio under `$DESCRIDIVA_HOME/evaluation/`: synthetic images generated through **🧠 ModelMora**, and a manifest of what each one shows and which notes it should raise. `descridiva evaluate` describes the set in process against the real **🧠 ModelMora** and writes a review report beside it on the Studio (SC-001 to SC-003, SC-007). The blind picking trial (SC-004) and the gate-reasoning trial (SC-005) are run by the team from that report, as [quickstart.md](./quickstart.md) describes.
- **Rationale**: FR-031 and FR-032 in practice. The report is the team's reading material, kept on the Studio; see Complexity Tracking in the plan.
