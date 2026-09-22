# **💬 DescriDiva**

> Image perception for **🖼️ MiraVeja**: the eyes of personas and the curator.

**Etymology:** "Descri" (from Portuguese "descrever": to describe) + "Diva" (glamorous performer).

## Boundaries

- **Owns:** turning an image into a neutral, detailed description others can reason about.
- **Never owns:** judgment, persona voice.
- **Runs on:** the Studio.

## Contracts and dependencies

- **Consumes:** **🧠 ModelMora** (vision and text inference).
- **Consumed by:** **🎭 SonaVida**, **🧐 CuraGusta**.

## Constitution rules that bite hardest

- **V:** open-weight models only, through **🧠 ModelMora**.
- **Component rule:** neutral descriptions; no persona voice, no verdicts.

## Repository

- Repository: `JomarJunior/descridiva` (public, Apache-2.0). Local: `components/descridiva/`.
- The earlier DescriDiva is archived privately as `JomarJunior/descridiva-legacy`. Its code is not carried over.
- To do in the first feature: README headed **💬 DescriDiva** linking to the hub; persona-definition and secrets check.

## Specs

005 `descridiva-perception`. Prompt in `docs/foundation/SPEC-ROADMAP.md`.
