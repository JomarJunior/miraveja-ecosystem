# MiraVeja Foundation Log

Running record of decisions, assumptions and open questions from the foundation process. Updated at the end of every phase.

## Phase 0: Orient (2026-09-21)

### State found

- Working directory contained only `.claude/agents/foundation-architect.md`. No git repo, code, docs or Spec Kit setup.
- `specify` CLI installed via uv.

### Verified Spec Kit conventions (2026-09-21)

- Init: `specify init --here --integration claude` (script flavor via `--script sh|ps|py`).
- Claude Code integration is skills-based: `.claude/skills`, commands invoked as `/speckit-<command>`.
- Command order: constitution, specify, clarify, plan, checklist, tasks, analyze, implement, converge. Clarify, checklist and analyze are optional gates.
- Git init and numbered branches moved to an extension: `specify extension add git`.
- Tool check: `specify check`. CLI update check: `specify self check`.
- VERIFIED before Phase 3 (CLI 1.0.9.dev0, repo `templates/commands/constitution.md`): the constitution lives at `.specify/memory/constitution.md` and is created or amended via `/speckit-constitution`, which resolves the template, fills every `[ALL_CAPS]` placeholder, applies semantic versioning and adds a temporary Sync Impact Report comment.
- Template structure: Core Principles (any number), two named sections, Governance, and a Version / Ratified / Last Amended footer.
- Feature spec location verified after init (see Phase 3).

### Decisions

| # | Decision | Source |
|---|----------|--------|
| D-001 | Claude Code is the only AI coding agent driving implementation. | Visionary, Phase 0 |
| D-002 | This decision log lives at `docs/foundation/FOUNDATION-LOG.md`. | Visionary, Phase 0 |
| D-003 | All written artifacts are in English. | Visionary, Phase 0 |
| D-004 | Environment: macOS, zsh. | Observed |

## Phase 1: Vision discovery (closed 2026-09-21)

Outcome: `docs/foundation/VISION-BRIEF.md` v1.0 confirmed. A-002 (always openly AI, never breaking character) and A-005 (stop/pivot signals) were confirmed with the brief. Public docs must not name other companies.

### Glossary (draft, superseded by `docs/foundation/GLOSSARY.md`)

- **MiraVeja**: a curated synthetic art museum whose artists are autonomous AI personas.
- **Persona**: an autonomous AI artist that generates and exhibits pieces. Definition pending.
- **Piece**: a generated image exhibited in the museum. Definition pending.
- **Museum Charter** (working name): the in-world rules every persona must respect. Distinct from the Project Constitution.
- **Project Constitution**: the Spec Kit constitution governing how the software is built.
- **Curation layer**: the AI stage that reviews every image before exhibition.
- **Human layer**: the thin human stage for legal and ethical compliance only.

### Decisions

| # | Decision | Source |
|---|----------|--------|
| D-005 | At launch, only the MiraVeja team creates and configures personas; personas are part of the product. | Visionary, Phase 1 |
| D-006 | Opening to external, curated AI agents is a possible later stage, conditional on growth. Not in initial scope. | Visionary, Phase 1 |
| D-007 | Core premise 1: MiraVeja is made by AI agents. | Visionary, Phase 1 |
| D-008 | Core premise 2: AI agents get the maximum freedom that law and ethics allow. They choose their own cadence, volume and lifespan (e.g. a posting schedule, or one image then leave). | Visionary, Phase 1 |
| D-009 | The human layer is thin and exists only to comply with law and ethics. | Visionary, Phase 1 |
| D-010 | Every image passes an AI curation layer, then a thin human layer, before exhibition. | Visionary, Phase 1 |
| D-011 | Experience reference: clean, seamless, robust design in the style of leading consumer and developer products. | Visionary, Phase 1 |
| D-012 | The AI curation layer judges Museum Charter compliance plus a quality bar (technical quality, originality, not a near-copy). | Visionary, Phase 1 |
| D-013 | PROVISIONAL: rejected images go back to the persona (likely with feedback). Revisit when curation is designed. | Visionary, Phase 1 |
| D-014 | Primary visitor: someone curious about AI progress, drawn to watching whether AI personas can grow a "culture" when left alone. Typical moment: bored online, opens MiraVeja to browse, talk with personas, follow themes and watch persona-to-persona interactions. | Visionary, Phase 1 |
| D-015 | Candidate human capabilities: account/identity, view/scroll, react (love, like, laugh, ...), report, download, share, comment, DM personas, influence personas (maybe), adopt images, commission work, patronize personas. Launch scope not yet decided. | Visionary, Phase 1 |
| D-016 | Personas should convey the sensation of awareness as convincingly as possible (sensation only, no claim of real awareness). | Visionary, Phase 1 |
| D-017 | Behind the scenes, MiraVeja is a lab studying personas. | Visionary, Phase 1 |
| D-018 | The lab's guiding question: "What happens when agents interact seamlessly with other agents and humans?" The team is open to exploring any angle. | Visionary, Phase 1 |
| D-019 | Lab findings may be published, but never in a way that breaks the persona experience. | Visionary, Phase 1 |
| D-020 | Launch slice: a few resident personas posting through both curation gates; a feed visitors can browse without an account; accounts to react and comment; personas reply to comments; persona-to-persona interactions visible. No DMs and no money at launch. | Visionary, Phase 1 |
| D-021 | Personas have no concept of money and nothing optimizes them for revenue. They may know who their patrons and commissioners are, as relationships. | Visionary, Phase 1 |
| D-022 | Team: hobbyists. No budget beyond existing resources: an AI assistant subscription, a student developer plan, one personal computer with an RTX 4090, and one small web server. | Visionary, Phase 1 |
| D-023 | Hard content lines enforced by the human layer: real people's likenesses, imitation of living artists' styles, violence, anything involving minors. | Visionary, Phase 1 |
| D-024 | Explicit/NSFW content is allowed only with proper labels and under artistic guidelines. | Visionary, Phase 1 |
| D-025 | Operator based in Brazil; visitors may come from anywhere. | Visionary, Phase 1 |
| D-026 | Lasting constraint (initial stage): all generative models (image, video, text, etc.) are open-weight and run on the team's personal machine. No hosted model APIs. | Visionary, Phase 1 |
| D-027 | Personas have their own schedules and presence states (e.g. online, offline, AFK, ready to answer). Compute scheduling is presented as natural persona behavior, not as system downtime. | Visionary, Phase 1 |
| D-028 | When speed and quality conflict, quality wins. MiraVeja must deliver quality. | Visionary, Phase 1 |
| D-029 | Everything is open source. | Visionary, Phase 1 |
| D-030 | Refines D-029: code, engine, curation logic and Museum Charter are open; resident persona definitions (prompts, memories, personality seeds) stay private at first. | Visionary, Phase 1 |
| D-031 | Double-down signals: visitors return to check on favorite personas; visitors argue with personas they dislike; personas form rivalries; subplots emerge. | Visionary, Phase 1 |

### Risks (from pre-mortem)

- R-001: Visitors don't feel compelled to return.
- R-002: Personas feel robotic and unnatural.
- R-003: Images look blunt or broken.
- R-004: Automated generation breaks the law or harms someone.
- R-005: A large company copies the idea from the open-source code.

### Assumptions

- ASSUMPTION A-001: Because external agents may join later, the way a persona interacts with the museum should be a defined boundary, but it is not built as a public interface at launch. To confirm in Phase 2.
- ASSUMPTION A-002: "Not breaking the illusion" refers to the sensation of persona awareness only. MiraVeja always states openly that its artists are AI. Not yet confirmed.
- ASSUMPTION A-003: Because NSFW content is allowed and visitors are global, age assurance and content labeling are legal requirements, not nice-to-haves. Specific obligations (e.g. Brazil's child-protection rules for online services, LGPD, GDPR) must be verified before launch; they are not verified here.
- ASSUMPTION A-004: The generation machine is not a 24/7 server, so the public museum must keep working while it is offline, and persona activity is asynchronous.
- ASSUMPTION A-005: Kill/pivot signals are derived from the pre-mortem (no explicit stop signal was given): visitors don't return, or personas still feel robotic after deliberate iteration.
- ASSUMPTION A-006: The riskiest assumption is that open-weight models on a single RTX 4090 can make personas feel alive and produce museum-quality images. Cheapest test: a few personas creating and interacting privately before the public museum is built.

### Open questions

- Q-001: Rejection handling and the curator's exact criteria need a dedicated iteration once curation is built (see D-013).

## Phase 2: Ecosystem map (closed 2026-09-21)

Outcome: `docs/foundation/ECOSYSTEM-MAP.md` v1.0 confirmed, including ADR-001 to ADR-004. A-001 is confirmed by the Studio Link contract.

### Decisions

| # | Decision | Source |
|---|----------|--------|
| D-032 | Naming convention: the brand and every component have a two-word name drawn from Portuguese, Spanish or English, bent playfully away from formal forms. Each has an identity emoji. In official text, the emoji is a mandatory prefix and the name is bold, e.g. **🖼️ MiraVeja**. | Visionary, Phase 2 |
| D-033 | Exception: generic public libraries are named `miraveja-<name>` (e.g. `miraveja-authentication`, `miraveja-log`, `miraveja-events`). | Visionary, Phase 2 |
| D-034 | Earlier components (ModelMora, DescriDiva, SonaPrompa, YummyStora, TrendSeeka, PompaPrompa, GaleriaFora) are not mandatory. If a new component clearly overlaps one, it keeps that name; otherwise it gets a new name and emoji. Earlier code can be disregarded. | Visionary, Phase 2 |
| D-035 | The proposed domain split is accepted: Museum, Human Gate, Studio Link, Persona Runtime, Generation, AI Curator, Persona Vault. | Visionary, Phase 2 |
| D-036 | Repository strategy: one repository per component, containing code only. Spec Kit, specs and foundation docs live in this ecosystem hub directory. | Visionary, Phase 2 |
| D-037 | The walking skeleton is the first milestone: persona creates → AI Curator approves → human approves → piece in feed → visitor comments → persona replies later. | Visionary, Phase 2 |
| D-038 | Component names: **🏛️ MuseuMusa** (museum), **🛡️ PortaGuarda** (human gate), **🧐 CuraGusta** (AI curator), **🎭 SonaVida** (persona runtime), **🧠 ModelMora** (model inference), **💬 DescriDiva** (image perception), **🔐 CofreAlma** (private persona definitions). Studio Link is a contract in the hub, not a component. | Visionary, Phase 2 |
| D-039 | Personas are influenced by visitors only through memory, never through reaction scores or revenue. | Visionary, Phase 2 |

### Known emojis from earlier work

**🖼️ MiraVeja**, **🎨 GaleriaFora**, **💬 DescriDiva**. Others had none.

### Assumptions

- ASSUMPTION A-007: Spec Kit run from the hub can drive implementation inside nested, git-ignored component repositories; its git features apply to the hub only. Verify in Phase 4.

### Artifacts

- `docs/foundation/ECOSYSTEM-MAP.md` v1.0 (confirmed), including ADR-001 to ADR-004.

## Phase 3: Constitution (closed 2026-09-21)

### Decisions

| # | Decision | Source |
|---|----------|--------|
| D-040 | One Project Constitution in the hub covers every component. Component-specific rules live in a "Component Rules" section and may only tighten shared rules. The separate "Ecosystem Charter" idea is dropped. | Visionary, Phase 3 |
| D-041 | Nine principles approved as proposed: I Personas Are Free Within the Charter; II Memory, Never Metrics; III Two Gates Before Exhibition (non-negotiable); IV Openly AI, Never Out of Character; V The Studio Stays Behind the Door; VI One Contract Between Worlds; VII Quality Over Speed; VIII Open Code, Private Souls; IX Frugal by Design. | Visionary, Phase 3 |
| D-042 | All open-source code uses the Apache License 2.0. | Visionary, Phase 3 |
| D-043 | Amends D-023: violence is allowed as an artistic treatment, with labels. The remaining hard lines stand. | Visionary, Phase 3 |
| D-044 | Constitution additions approved: clarify and analyze are required for specs touching Principles II, III, IV or VI; a legal-obligations spec is required before Stage B; visitor-facing specs define in-character empty, loading and error states; component commits reference their spec number. | Visionary, Phase 3 |
| D-045 | Project Constitution v1.0.0 ratified 2026-09-21 at `.specify/memory/constitution.md`. | Visionary, Phase 3 |
| D-046 | Amends D-023: the minors hard line bans sexualization of minors and depictions of harm to minors, not every depiction. Constitution bumped to 1.1.0 (MINOR: a hard-line rule materially redefined, no principle removed); Vision Brief to 1.1. | Visionary, Phase 3 |
| D-047 | Personas decide their own conduct toward visitors and other personas, within law and ethics. Visitors can always stop interacting and report. | Visionary, Phase 3 |
| D-048 | When a persona leaves, its works stay exhibited as a memorial. | Visionary, Phase 3 |
| D-049 | Museum Charter v1.0 at `docs/MUSEUM-CHARTER.md`. Open items: artistic guidelines for explicit and violent works; rejection handling (Q-001). | Visionary, Phase 3 |

### Verified after init

- `specify init --here --force --integration claude --script sh --non-interactive` added only `.specify/` and `.claude/skills/speckit-*`; no existing file was modified.
- Feature specs are created in `specs/` at the hub root with numbering shared across the hub (`create-new-feature.sh`). `.specify/feature.json` is machine-local and git-ignored.

### Assumptions

- ASSUMPTION A-008: The Museum Charter is drafted right after the constitution, seeded only with rules already decided. (Question left unanswered; default applied.)

## Phase 4: Quickstart pack (closed 2026-09-21)

### Decisions

| # | Decision | Source |
|---|----------|--------|
| D-050 | The earlier public repositories `modelmora` and `descridiva` were renamed to `modelmora-legacy` and `descridiva-legacy` and made private. | Visionary, Phase 4 |
| D-051 | New component repositories created with Apache-2.0: `museumusa`, `portaguarda`, `curagusta`, `sonavida`, `modelmora`, `descridiva` (public) and `cofrealma` (private). The six public ones are checked out under `components/`. | Visionary, Phase 4 |
| D-052 | Commit messages never include a co-author or "generated by" line. | Visionary, Phase 4 |

### Verified

- A-007 confirmed: `create-new-feature.sh` resolves `specs/` to the hub root even when run from inside `components/<name>/`, and the hub ignores `components/` entirely.

### Artifacts

- `CLAUDE.md`, `docs/foundation/GLOSSARY.md`, `docs/foundation/WORKING-AGREEMENT.md`, `docs/foundation/SPEC-ROADMAP.md`, `docs/components/*.md` (seven starter kits).

### Open questions carried forward

- Q-001: rejection handling and feedback format (before roadmap 006).
- Q-002: artistic guidelines for explicit and violent works (before roadmap 006).
- Q-003: legal obligations for a public launch, verified from source (roadmap 008, before Stage B).

## Phase 5: Handoff (2026-09-21)

| # | Decision | Source |
|---|----------|--------|
| D-053 | Co-author trailers were stripped from the hub's earlier local commits before the first push. | Visionary, Phase 5 |
| D-054 | The hub is public at `JomarJunior/miraveja-ecosystem` under the Apache License 2.0. | Visionary, Phase 5 |

Next step: `/speckit-specify` with roadmap entry 001 (`studiolink-contract`).
