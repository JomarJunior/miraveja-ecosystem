# Phase 0 Research: Resident Persona Definition Format

Decisions behind the plan. Those marked *(Visionary)* were chosen directly.

## R-1: File format *(Visionary)*

- **Decision**: one YAML file per persona, `definition.persona.yaml`, validated by a closed JSON Schema 2020-12 document (`additionalProperties: false` everywhere). Author's notes are separate YAML files, `*.note.yaml`, with their own closed schema.
- **Rationale**: most of a definition is prose a team member writes by hand (FR-009). YAML block scalars (`|`) hold paragraphs without escaping, diff well in review, and stay structured enough for seed memories and shared pasts to be lists of records. JSON Schema is the same schema language the Studio Link already uses (spec 001), so the hub has one way to state a format. A closed schema makes an unknown part a refusal, which is how FR-032 ("carries or points to an author's note") is enforced: there is simply no field for one.
- **Alternatives considered**: Markdown with front matter (the nicest to write, but seed memories and shared pasts would be headings parsed by convention, and validation would be loose); TOML (strict, but multi-line prose is awkward and nested lists of records read badly).

## R-2: Where the format's code lives *(Visionary)*

- **Decision**: a new public library, `miraveja-persona` (Python 3.12, Apache-2.0, repository `JomarJunior/miraveja-persona`, checked out at `components/miraveja-persona/`). It holds the bundled schemas, the loader, the check, the shared-past listing, the scaffold for new definitions and the public-repository guard, behind one CLI and a small Python API.
- **Rationale**: **🔐 CofreAlma** owns no code (component rule) and all component code is public (Principle VIII). Four different places need this code: **🎭 SonaVida** loads definitions; **🔐 CofreAlma**'s CI and the writer's machine check them; the lab lists shared pasts; every public repository's CI guards against leaks. Only a public library can be installed in all four. It is generic enough to follow the library naming convention (D-033), like `miraveja-studiolink`.
- **Alternatives considered**: code inside **🎭 SonaVida** (**🔐 CofreAlma** CI and every guard would install the whole runtime just to check a file); scripts in the hub (the hub holds rules and docs, and each repository would vendor a copy); private code in **🔐 CofreAlma** (needs a constitution amendment, and public repositories could not use it for their guard).

## R-3: Where the schema's source of truth lives

- **Decision**: the schemas live in this hub at `specs/003-cofrealma-persona-definition/contracts/`. The library ships a copy inside its package, and its CI fails if the copy differs from the hub file at the pinned hub commit.
- **Rationale**: the format is a hub asset like the Studio Link contract. Unlike `miraveja-studiolink`, which reads the hub checkout at runtime, the loader runs on the Studio inside **🎭 SonaVida**, where requiring a hub checkout just to read a persona is fragile. A bundled copy plus an equality test gives one source of truth without a runtime dependency on the hub.
- **Alternatives considered**: reading the hub checkout at runtime (as `miraveja-studiolink` does; fine for test tools, awkward for a runtime); the library as the source of truth (the format would stop being a hub document).

## R-4: How the check finds orders, metrics, feelings and human claims

- **Decision**: a deterministic, rule-based check with a catalog of named rules. Each rule is a small set of English patterns and word lists applied to the prose parts it covers, producing findings that are either **certain** or **uncertain** (FR-017). No model is involved.
- **Rationale**: FR-016 demands the check run on the writer's machine and send nothing anywhere. **🧠 ModelMora** answers only on the Studio's loopback, so a model-assisted check would not run on a laptop, and a hosted model is forbidden (Principle V). Deterministic rules are also reproducible, so SC-003, SC-007 and SC-008 can be measured on a labeled corpus and every finding can quote its trigger. Where words alone cannot decide ("still thinks of B" as fact or feeling), the rule reports *uncertain* instead of guessing.
- **Rule families** (full catalog in [contracts/check-rules.md](./contracts/check-rules.md)): prescribed cadence or volume (FR-010); prescribed subject (FR-011); hard lines (FR-012, FR-036); money and metrics (FR-013); visitors and Studio Link content (FR-014); secrets (FR-015); feelings or motives in a shared past (FR-026, FR-030); orders about future conduct (FR-030); human claims (FR-035); model names (FR-007); structure (FR-001 to FR-009, FR-027, FR-034).
- **Alternatives considered**: a local model through **🧠 ModelMora** (Studio-only, non-deterministic, and a finding could not always say what triggered it; may be revisited as an optional second opinion on the Studio); a pure schema check (cannot see meaning at all).

## R-5: Telling real people apart from fictional ones

- **Decision**: each definition declares its fictional **lore names**: the invented people, places, teams and works its prose mentions. The check treats every other proper name in prose (a capitalized word run not at a sentence start, not a common word, not the persona's own public name, not a declared lore name) as an *uncertain* finding: "is this a real person or a living artist?"
- **Rationale**: no offline tool can know every real person, and shipping a list of real people in a public repository would itself name them. Declaring what is invented turns the question around: the writer states what is fiction, and anything undeclared is asked about. It costs a few lines per definition and makes SC-003's "flags 100% of seeded hard-line content" achievable, since a seeded real name is always undeclared.
- **Alternatives considered**: a list of well-known names (incomplete, and public text must not name people or companies); no name check (FR-012 would rely on the gates alone).

## R-6: Recording the writer's decisions on uncertain findings

- **Decision**: uncertain findings do not fail the check by themselves, but they are listed and the command exits with a distinct code until each one is decided. A team member records a decision in `decisions.yaml` beside the definition in **🔐 CofreAlma**: the finding's fingerprint (rule, a stable part key that names list items by `id` or `story` rather than position, and a hash of the triggering words), *accepted* or *not an issue*, who decided, and when. A decided finding is then reported as decided, never silently dropped. Certain findings cannot be decided away; the definition must change.
- **Rationale**: FR-017 forbids both passing silently and blocking outright. A durable decision keeps CI reproducible and leaves a trail a reviewer can read.
- **Alternatives considered**: inline markers in the definition (would mix review state into the persona's own file, which a runtime reads); warnings only (would pass silently in CI).

## R-7: Shared pasts across definitions

- **Decision**: a shared past has a **story** key shared by every definition that tells it, a **participants** list of stable identifiers (including the persona's own), a rough **when**, the **happened** text, and a **telling**: `agreed` or `intended-difference`. The text names participants by position (`{1}`, `{2}`), never by name or point of view, so two definitions that agree can hold identical text; the runtime renders it from each persona's point of view.
- **Difference detection**: for one story, entries marked `agreed` in every definition must match exactly (participants, when, text). A mismatch is an *uncertain* finding, "possible mistake" (FR-029). Entries marked `intended-difference` in every definition pass with no warning. Mixed markings are a certain finding, since the team disagrees with itself. A story told in one definition only is a one-sided past and needs no counterpart.
- **Listing (FR-028)**: `miraveja-persona pasts <cofrealma-root>` prints every story with its participants (UUID and public name), its telling, and every version, as the baseline for spec 007.
- **Rationale**: exact comparison is the only deterministic way to tell "the same account" from "a different one" without reading meaning. Position placeholders make agreement expressible as identical text, so agreeing is cheap and differing is always a deliberate mark.
- **Alternatives considered**: free first-person text per definition (every pair would differ, so every story would need a mark or a warning); a single shared-story file referenced by definitions (a definition would no longer be self-sufficient, against User Story 2).

## R-8: Birth, freezing and reuse (FR-018, FR-037)

- **Decision**: **🔐 CofreAlma** keeps a birth ledger, `ledger/births.yaml`, with each born persona's identifier, public name, the SHA-256 of its definition file, and the birth date. `miraveja-persona freeze` adds an entry. The check fails if a frozen definition's file no longer matches its hash, if a new definition reuses a ledger identifier or public name, or if a ledger entry's file is gone. The loader brings a resident persona to life only if its definition matches a ledger entry. Anything the team wants to add after birth goes into an author's note, which no runtime reads.
- **Rationale**: FR-037 wants the original kept unchanged for good and later edits kept apart; a hash in a ledger makes "unchanged" checkable, and author's notes already exist as the place for things the persona must not see. Requiring a ledger entry at load time means no persona can come alive without its beginning on record.
- **Alternatives considered**: git history as the record (history can be rewritten, and the Studio should not need git to load a persona); a revisions folder for edited copies (a second definition-shaped file is exactly what a runtime might load by mistake).

## R-9: Recognizing definitions in public repositories (FR-021)

- **Decision**: `miraveja-persona guard [paths]` blocks, in any tracked file: a YAML document whose top level has the signature key `miravejaPersona` or `miravejaAuthorNote`; any file named `*.persona.yaml` or `*.note.yaml`; the **🔐 CofreAlma** root marker file `.cofrealma`; and any line in any text file beginning with the signature key (a pasted fragment). It allows only documents that also say `nature: synthetic`. Its output names the file, line and reason, and **never quotes content**.
- **Rationale**: the signature key is required by the schema, so every real definition carries it. Blocking unmarked or unrecognized markings as resident follows FR-021 and User Story 4 scenario 2. Not quoting means a leak is never copied into a public CI log.
- **Residual risk**: prose copied out of a definition without its keys cannot be recognized by any format check. The existing per-repository guards (spec 001 T005, spec 002 T004) still block resident public names in fixtures; Principle VIII's review duty covers the rest.
- **Alternatives considered**: hashed lists of resident names shared with public CI (short names can be recovered from hashes by guessing, so the list would leak them).

## R-10: Resident definitions only on the Studio (FR-022)

- **Decision**: the loader has two entry points. `load_resident(path, cofrealma_root)` requires `nature: resident`, a path inside a directory that carries the `.cofrealma` marker, and a matching birth ledger entry; it refuses everything else. `load_synthetic(path)` requires `nature: synthetic` and is for tests and examples. Both refuse author's notes and any unknown field.
- **Rationale**: the only place a `.cofrealma` marker legitimately exists is the private checkout on the Studio (the guard blocks it anywhere public), so "inside a marked **🔐 CofreAlma** checkout" is a practical stand-in for "on the Studio". Two entry points make it impossible to bring a synthetic persona to life as a resident by passing a flag.
- **Known limit**: a copy of the vault checkout on another team machine would also pass `load_resident`. Keeping the vault on the Studio alone is a team practice, recorded in the spec's Assumptions; the guard still keeps it out of every public repository.
- **Alternatives considered**: an environment variable declaring the Studio (trivially set anywhere); a Studio-only config file (the same weakness, one more file to keep private); checking the git remote (the Studio should not need git to load a persona).

## R-11: Writing a definition in under an hour (SC-001)

- **Decision**: `miraveja-persona new --name "<public name>"` writes a scaffold with a fresh UUID, every required and optional part, and one-line guidance for each, using the synthetic example's style. The synthetic example lives in the hub at `contracts/examples/`.
- **Rationale**: the hour goes to writing the persona, not to learning the structure or generating identifiers. The guidance lines are YAML comments, so they are dropped on load and never reach a runtime.
- **Alternatives considered**: a blank template file to copy (no fresh identifier, and easy to reuse the example's by mistake, which the check would then catch).
