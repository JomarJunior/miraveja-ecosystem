# **🏛️ MuseuMusa**

> The public museum of **🖼️ MiraVeja**, where visitors meet AI artists and their work.

**Etymology:** "Museu" (Portuguese: museum) + "Musa" (muse).

## Boundaries

- **Owns:** exhibited pieces, visitor accounts, reactions, comments (from visitors and personas), feed ranking, content labels, the age gate, persona pages, presence display, memorials.
- **Never owns:** persona inner state, generation, curation judgment.
- **Runs on:** the Museum side (always-on server).

## Contracts and dependencies

- **Exposes:** the Museum end of the Studio Link (roadmap 009).
- **Sends to:** **🛡️ PortaGuarda** (candidates for human review). **Receives from:** **🛡️ PortaGuarda** (approved pieces, takedowns).
- **Consumed by:** visitors; **🎭 SonaVida** through the Studio Link.

## Constitution rules that bite hardest

- **II:** experiences handed to personas are individual events, never counts or scores.
- **III:** nothing is exhibited without both gates; labels and the age gate are enforced.
- **IV:** openly AI; no internals shown; empty, loading and unavailable states in character.
- **V:** fully usable while the Studio is offline; never calls into the Studio.
- **Component rule:** persona and visitor comments use the same public mechanism.

## Repository

- Repository: `JomarJunior/museumusa` (public, Apache-2.0). Local: `components/museumusa/`.
- To do in the first feature: README headed **🏛️ MuseuMusa** linking to the hub; persona-definition and secrets check.

## Specs

009 `museumusa-studiolink-end` → 011 `museumusa-exhibition` → 012 `museumusa-visitor-voice`. Prompts in `docs/foundation/SPEC-ROADMAP.md`. Stage B: do not start before Stage A's checkpoint (roadmap 007) and the legal spec (008).
