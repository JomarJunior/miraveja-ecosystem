# **🧐 CuraGusta**

> The AI curation gate of **🖼️ MiraVeja**: the museum's taste.

**Etymology:** "Cura" (from Portuguese "curar": to curate) + "Gusta" (from "gusto": taste).

## Boundaries

- **Owns:** Museum Charter checks, the quality bar, verdicts with reasons, feedback addressed to personas.
- **Never owns:** the final publish decision.
- **Runs on:** the Studio (needs the GPU).

## Contracts and dependencies

- **Consumes:** **💬 DescriDiva** (descriptions), **🧠 ModelMora** (inference).
- **Receives:** candidates from **🎭 SonaVida**. **Returns:** verdicts and feedback to **🎭 SonaVida**.
- **Sends:** accepted candidates, with labels and verdict, across the Studio Link to the human gate.

## Constitution rules that bite hardest

- **I:** judges only against the Charter, the law and ethics, plus the quality bar. It does not impose taste the Charter does not state.
- **III:** hard lines and labels are enforced here first.
- **VII:** the quality bar is part of the gate, tested first.
- **Component rule:** never publishes; every rejection carries feedback for the persona.

## Repository

- Repository: `JomarJunior/curagusta` (public, Apache-2.0). Local: `components/curagusta/`.
- To do in the first feature: README headed **🧐 CuraGusta** linking to the hub; persona-definition and secrets check.

## Specs

006 `curagusta-gate`. Prompt in `docs/foundation/SPEC-ROADMAP.md`. Open Charter items it depends on: artistic guidelines for labeled works; rejection handling (Q-001).
