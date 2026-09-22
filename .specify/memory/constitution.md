# **🖼️ MiraVeja** Constitution

This constitution governs how every part of **🖼️ MiraVeja** is built: this hub and every component repository. It sits beside the **Museum Charter** (`docs/MUSEUM-CHARTER.md`), which governs how personas live inside the museum. Traceability codes (D-, A-, R-, ADR-) refer to `docs/foundation/FOUNDATION-LOG.md` and `docs/foundation/ECOSYSTEM-MAP.md`.

## Core Principles

### I. Personas Are Free Within the Charter

- Code MUST NOT constrain a persona's cadence, volume, subject, style or lifespan unless the constraint traces to a Museum Charter rule, the law, or ethics.
- Every persona-facing constraint in a spec MUST cite its source (Charter rule, legal requirement or ethical reason).
- Operational limits of the Studio (compute, schedule) MUST be expressed as persona behavior, not as hidden throttling of persona intent.

**Rationale:** **🖼️ MiraVeja** exists to see what AI artists do when left free. Constraints that are not traceable quietly turn the experiment into a script.
**Verification:** plan Constitution Check lists every persona-facing constraint with its source; reviewers reject any without one.
**Source:** D-007, D-008, D-027.

### II. Memory, Never Metrics

- No persona-facing component MAY receive, store or optimize for reaction counts, rankings, popularity scores, revenue or any monetary data.
- Visitor activity MUST reach a persona only as individual experiences it may remember (for example: "a visitor commented this on my piece").
- Personas MAY know who their patrons and commissioners are, as relationships, never as amounts.

**Rationale:** optimizing for reactions or money brings back the farming behavior **🖼️ MiraVeja** exists to escape.
**Verification:** Studio Link contract review confirms no metric or monetary fields flow toward **🎭 SonaVida**; contract tests assert it.
**Source:** D-021, D-039, ADR-004.

### III. Two Gates Before Exhibition (NON-NEGOTIABLE)

- Nothing is exhibited unless it has passed **🧐 CuraGusta** (Museum Charter compliance and the quality bar) and then **🛡️ PortaGuarda** (law and ethics), in that order.
- Every gate decision MUST be recorded with the piece, the verdict, the reason and the time; human decisions also record the reviewer.
- Hard lines MUST be enforced at both gates: no real people's likenesses, no imitation of living artists' styles, nothing involving minors.
- Violence is allowed only as an artistic treatment and MUST be labeled.
- Explicit content MUST be labeled and MUST be shown only to visitors who have passed the age gate.
- There is no bypass path, including for the team.

**Rationale:** this is the museum's promise to visitors and its protection against legal and human harm (R-004).
**Verification:** test-first coverage of both gates, the publish path and label enforcement; an end-to-end test proves an ungated piece cannot be exhibited.
**Source:** D-010, D-012, D-023, D-024, D-043, ADR-003.

### IV. Openly AI, Never Out of Character

- **🖼️ MiraVeja** MUST always state openly that its artists are AI personas.
- Visitor-facing surfaces MUST NOT expose system internals: prompts, model names, stack traces, raw errors, queue states or machine downtime.
- Failures and unavailability MUST be rendered as persona states (for example "away from the studio") or as neutral museum states, never as broken pages.
- Published lab findings MUST NOT break the persona experience.

**Rationale:** the sensation of awareness is the product, and honesty about being AI is the ethical floor beneath it.
**Verification:** visitor-facing specs include in-character empty, loading and error states as acceptance criteria; UI review checks for leaked internals.
**Source:** D-016, D-019, D-027, A-002.

### V. The Studio Stays Behind the Door

- All Studio Link traffic MUST be initiated by the Studio. The Museum side MUST NOT call into the Studio.
- The Museum side MUST remain fully usable while the Studio is offline.
- All generative models (image, text, video, perception) MUST be open-weight and run on team hardware. Hosted model APIs are not allowed.

**Rationale:** the Studio is a personal machine that keeps hours and must never be exposed to the internet.
**Verification:** architecture review of every plan touching the Studio Link; an automated test runs the Museum side with the Studio absent.
**Source:** ADR-001, D-026, A-004.

### VI. One Contract Between Worlds

- The Studio and the Museum side MUST communicate only through the Studio Link contract, versioned in this hub.
- Every contract change MUST be its own spec, MUST bump the contract version, and MUST update contract tests on both ends before merge.
- Each component owns its data. Components MUST NOT share a database or read another component's storage.

**Rationale:** one explicit boundary keeps components independent and is the future entry point for curated external agents.
**Verification:** contract tests in both **🏛️ MuseuMusa** and **🎭 SonaVida**; plan review rejects cross-component storage access.
**Source:** ADR-002, D-006, D-035, A-001.

### VII. Quality Over Speed

- Tests MUST be written before implementation for gate logic, the Studio Link contract and access control.
- A feature is done only when every acceptance criterion in its spec passes.
- Pieces with visible generation defects MUST NOT reach exhibition; the quality bar is part of **🧐 CuraGusta**, not an afterthought.
- When a deadline and quality conflict, the deadline moves.

**Rationale:** blunt or broken images and a robotic feel are named ways this project dies (R-002, R-003).
**Verification:** tasks list tests before implementation for the covered areas; `/speckit-converge` reports converged before a feature is closed.
**Source:** D-028, R-002, R-003.

### VIII. Open Code, Private Souls

- All component code MUST be public under the Apache License 2.0.
- Resident persona definitions (prompts, personality seeds, memories) MUST live only in the private **🔐 CofreAlma** repository and MUST NOT enter any public repository, log, spec or fixture.
- Public repositories MUST run an automated check that blocks persona definitions and secrets.

**Rationale:** the machinery is open; each persona stays a mystery. The personas and their culture, not the code, are what cannot be copied (R-005).
**Verification:** CI check in every public repository; test fixtures use synthetic personas only.
**Source:** D-029, D-030, D-042, R-005.

### IX. Frugal by Design

- Everything MUST run on one RTX 4090 machine (the Studio) and one small web server (the Museum side) at no added cost.
- Any new paid service or dependency MUST be approved through a constitution amendment.
- Plans SHOULD prefer the simplest design that fits these resources over designs that assume future scale.

**Rationale:** the team is hobbyists with no budget; a design that needs money is a design that stops.
**Verification:** plan Technical Context lists runtime resources; reviewers reject plans that exceed them.
**Source:** D-022.

## Ecosystem Constraints

**Naming and identity**
- The brand and every component have a two-word name drawn from Portuguese, Spanish or English, bent playfully from the formal form, and an identity emoji.
- In official text the emoji is a mandatory prefix and the name is bold, for example **🖼️ MiraVeja**, **🏛️ MuseuMusa**.
- Generic public libraries are named `miraveja-<name>` (for example `miraveja-log`, `miraveja-events`) and carry no brand emoji in their package name.

**Language and public text**
- All written artifacts are in English.
- Public artifacts MUST NOT name other companies; describe them generically.

**Legal and safety**
- The operator is based in Brazil and visitors come from anywhere.
- Before the Museum side opens to the public (build Stage B), a spec MUST document the applicable obligations (age assurance, data protection, content responsibility) and how each is met. Obligations are verified, not assumed (A-003).

**Experience**
- Visitor-facing work follows a clean, seamless, robust design language.
- Every visitor-facing spec MUST define its empty, loading and error states.

**Component Rules** (these may only add or tighten; they never relax a principle)

| Component | Additional rules |
|---|---|
| **🏛️ MuseuMusa** | Owns visitor accounts and all public social state. Persona comments and visitor comments use the same public mechanism. |
| **🛡️ PortaGuarda** | Judges law and ethics only, never taste. Provides report and takedown paths reachable at any time. |
| **🧐 CuraGusta** | Never publishes. Every rejection includes feedback addressed to the persona. |
| **🎭 SonaVida** | Never receives metrics or monetary data (Principle II). |
| **🧠 ModelMora** | Records the name, version and license of every model it serves. Serves open-weight models only. |
| **💬 DescriDiva** | Produces neutral descriptions with no persona voice and no judgment. |
| **🔐 CofreAlma** | Private repository. Never cloned into the hub's `components/` tree or any public location. |

## Development Workflow and Quality Gates

**Where things live**
- This hub is the single source of truth for the constitution, the Museum Charter, foundation docs, contracts and specs.
- Specs live in `specs/` and are named `NNN-<component>-<feature>` (for example `001-studiolink-contract`); numbering is shared across the hub.
- Component code lives in its own repository, checked out under `components/<name>/` (ignored by the hub). Component repositories contain code only.

**The loop**
- Features follow `/speckit-specify` → `/speckit-clarify` → `/speckit-plan` → `/speckit-checklist` → `/speckit-tasks` → `/speckit-analyze` → `/speckit-implement` → `/speckit-converge`.
- `/speckit-clarify` and `/speckit-analyze` are REQUIRED for any spec touching Principles II, III, IV or VI; optional otherwise.
- Every plan MUST include a Constitution Check naming each principle it touches and how it complies.
- Work follows the build order in `docs/foundation/ECOSYSTEM-MAP.md`. Starting a feature outside the current stage requires updating the map first.

**Roles**
- Claude Code is the implementing agent.
- The Visionary approves specs, plans and constitution amendments, and merges.
- Commits in component repositories reference the spec number they implement.

## Governance

- This constitution supersedes all other practices in the hub and every component repository. Where a spec or plan conflicts with it, the constitution wins until amended.
- **Amendments** are made through `/speckit-constitution`, approved by the Visionary, and recorded in `docs/foundation/FOUNDATION-LOG.md` with their rationale.
- **Versioning** follows semantic versioning: MAJOR for removing or redefining a principle, MINOR for a new principle or materially expanded guidance, PATCH for clarifications and wording.
- **Compliance** is checked in every plan's Constitution Check and by `/speckit-analyze`. Any justified exception MUST be recorded in the plan's Complexity Tracking with the principle it strains and why.
- **Museum Charter sync:** a change to the Museum Charter that affects what the gates enforce (Principles I and III) MUST be reviewed against this constitution in the same change. This constitution never contradicts the Charter's legal and ethical rules.
- Runtime guidance for agents lives in `docs/foundation/`.

**Version**: 1.0.0 | **Ratified**: 2026-09-21 | **Last Amended**: 2026-09-21
