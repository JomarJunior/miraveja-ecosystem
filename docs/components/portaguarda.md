# **🛡️ PortaGuarda**

> The thin human gate of **🖼️ MiraVeja**, for law and ethics.

**Etymology:** "Porta" (Portuguese: door) + "Guarda" (guard).

## Boundaries

- **Owns:** the human review queue, visitor reports, takedowns, the audit trail of every human decision.
- **Never owns:** taste or quality judgment (that is **🧐 CuraGusta**).
- **Runs on:** the Museum side (always-on server), reachable at any time.

## Contracts and dependencies

- **Receives:** candidates that passed the AI gate, via **🏛️ MuseuMusa**'s end of the Studio Link; visitor reports from **🏛️ MuseuMusa**.
- **Sends:** approvals and takedowns to **🏛️ MuseuMusa**.
- **Used by:** team moderators.

## Constitution rules that bite hardest

- **III (non-negotiable):** there is no path to exhibition that skips this gate, including for the team. Every decision records who, when and why.
- **Component rule:** judges law and ethics only; report and takedown paths are always available.

## Repository

- Repository: `JomarJunior/portaguarda` (public, Apache-2.0). Local: `components/portaguarda/`.
- To do in the first feature: README headed **🛡️ PortaGuarda** linking to the hub; persona-definition and secrets check.

## Specs

010 `portaguarda-review`. Prompt in `docs/foundation/SPEC-ROADMAP.md`. Stage B.
