# Data Model: Resident Persona Definition Format

The authoritative shapes are the schemas in [contracts/](./contracts/). This page explains them and the rules a schema cannot state. "Prose" means a plain-language string a team member writes; every prose field is checked by the rules in [contracts/check-rules.md](./contracts/check-rules.md).

## Persona definition (`definition.persona.yaml`)

| Field | Type | Required | Rules |
|---|---|---|---|
| `miravejaPersona` | integer, `1` | yes | Format major version and the signature the guard looks for (FR-001, FR-019, FR-021). |
| `nature` | `resident` \| `synthetic` | yes | FR-001. Only `synthetic` may appear in public (FR-021). |
| `identity.id` | UUID string | yes | Stable identifier, the same value as `personaId` in the Studio Link (spec 001). Never reused (FR-037). |
| `identity.publicName` | string, 1–80 | yes | Unique across all definitions and the birth ledger, departed personas included (FR-037). Not a real person or living artist (FR-012). |
| `identity.about` | prose | yes | Who the persona is, in the team's words (FR-002). |
| `identity.openlyAI` | `true` (constant) | yes | The persona knows it is an AI (FR-034). A definition without it is incomplete. |
| `identity.selfUnderstanding` | prose | no | How the persona holds its past as its own story while knowing it is an AI (FR-034). |
| `taste.drawnTo` | prose | yes | FR-003. |
| `taste.themes` | list of prose, 1+ | yes | Subjects and moods it keeps returning to. Inclinations, never rules (FR-011). |
| `taste.stylesAndMedia` | prose | yes | Favored styles and media, in words. No model names (FR-007). |
| `taste.dislikes` | prose | no | FR-003. |
| `voice.speech` | prose | yes | How it speaks and writes (FR-004). |
| `voice.temperament` | prose | yes | How it tends to treat visitors and other personas (FR-004, Charter Article 7). |
| `tendencies.presence` | prose | yes | When it tends to be in the studio, away or resting. A habit, never a timetable (FR-005, FR-010). |
| `tendencies.work` | prose | yes | How it tends to work. Never a frequency, count or deadline (FR-005, FR-010). |
| `cares` | list of prose, 1+ | yes | FR-006. |
| `craft` | prose | no | Craft preferences in words; no model, version or Studio internal (FR-007). |
| `selfImage` | prose | no | How it imagines itself looking; never a real person (FR-036). |
| `lore.names` | list of *Lore name* | no | Every invented proper name the prose uses (R-5). |
| `seedMemories` | list of *Seed memory*, 1+ | yes | FR-008. |
| `sharedPasts` | list of *Shared past* | no | FR-025 to FR-030. |

No other field is allowed. In particular there is no field for an author's note, a model, a schedule, a count or a visitor (FR-007, FR-010, FR-013, FR-014, FR-032).

### Lore name

| Field | Type | Required | Rules |
|---|---|---|---|
| `name` | string, 1–80 | yes | An invented name used in the prose. |
| `is` | `person` \| `place` \| `group` \| `work` \| `other` | yes | What it names. |

A lore name MUST NOT be the name of a real person or living artist. Declaring it is the writer's statement that it is invented (R-5).

### Seed memory

| Field | Type | Required | Rules |
|---|---|---|---|
| `id` | slug, `^[a-z0-9-]{1,64}$` | yes | Unique within the definition; lets an author's note point at it. |
| `when` | prose, 1–200 | yes | Roughly when in the persona's past ("as a child", "the winter before the museum"). |
| `happened` | prose | yes | What happened, from the persona's point of view. May be human-shaped (FR-035). May be a past act of concealment (FR-030). Never a visitor (FR-014). |

### Shared past

| Field | Type | Required | Rules |
|---|---|---|---|
| `story` | slug | yes | The same key in every definition that tells this past. Unique within one definition. |
| `participants` | list of UUID, 2+ | yes | Stable identifiers, including this persona's own. Every other identifier must have a definition (FR-027). |
| `when` | prose, 1–200 | yes | Roughly when. |
| `happened` | prose | yes | Past facts only, naming participants as `{1}`, `{2}`… by their position in `participants`. No feeling, no reason for a feeling, no order about future conduct (FR-026, FR-030). |
| `telling` | `agreed` \| `intended-difference` | yes | FR-029, R-7. |

**Across definitions** (checked with `--tree`, R-7):

| Situation | Result |
|---|---|
| Story in one definition only | One-sided past. Passes. |
| All entries `agreed`, identical `participants`, `when`, `happened` | Passes. |
| All entries `agreed`, any of them differ | Uncertain finding: possible mistake. |
| All entries `intended-difference` | Passes, listed as an intended difference. |
| Mixed `agreed` and `intended-difference` | Certain finding: the markings disagree. |
| Participant lists differ between entries | Certain finding, unless all are `intended-difference`. |

## Author's note (`*.note.yaml`)

| Field | Type | Required | Rules |
|---|---|---|---|
| `miravejaAuthorNote` | integer, `1` | yes | Signature and version (FR-033). |
| `nature` | `resident` \| `synthetic` | yes | As for definitions. |
| `about.persona` | UUID | no | The persona the note concerns. |
| `about.story` | slug | no | A shared past the note concerns. |
| `about.seedMemory` | slug | no | A seed memory; requires `about.persona`. |
| `truth` | prose | yes | What really happened, motives and feelings included (FR-031). |
| `writtenBy` | string | yes | The team member. |
| `writtenOn` | date | yes | |

At least one of `about.persona` or `about.story` is required. Hard-line, money, visitor and secret rules apply to `truth` (FR-033); the feeling and order rules do not, since no persona ever reads it (FR-032).

## Birth ledger (`ledger/births.yaml` in **🔐 CofreAlma**)

| Field | Type | Rules |
|---|---|---|
| `id` | UUID | One entry per born persona. Never removed (FR-037). |
| `publicName` | string | Never reused. |
| `definition` | path, relative to the **🔐 CofreAlma** root | The frozen file. |
| `sha256` | hex string | The file's hash at birth. The check fails if the file no longer matches (R-8). |
| `bornOn` | date | |

## Check decisions (`decisions.yaml` beside a definition)

| Field | Type | Rules |
|---|---|---|
| `finding` | fingerprint: rule id, part path, SHA-256 of the triggering words | Identifies one uncertain finding (R-6). |
| `decision` | `accepted` \| `not-an-issue` | `accepted` means the writer judged the words fine as written. |
| `by` | string | Team member. |
| `on` | date | |

Only uncertain findings can be decided. A decision whose fingerprint no longer matches any finding is reported as stale.

## Finding (check output)

| Field | Meaning |
|---|---|
| `rule` | Rule id from the catalog, e.g. `order.cadence`. |
| `part` | JSON Pointer to the field, e.g. `/sharedPasts/0/happened`. |
| `line` | Line in the file. |
| `quote` | The triggering words. Printed by `check`, never by `guard`. |
| `cites` | The requirement and Charter article or principle broken. |
| `certainty` | `certain` or `uncertain`. |
| `decided` | The recorded decision, if any. |

## Lifecycle of a definition

```text
draft ──check passes──► ready ──freeze (ledger entry)──► born ──persona leaves──► departed
  ▲                       │                                │
  └──── edits ────────────┘                                └─ file frozen for good; notes may be added beside it
```

- **draft / ready**: editable. `ready` means the check passes with every uncertain finding decided.
- **born**: `freeze` records the hash; `load_resident` now accepts it; the file may never change (FR-018, FR-037).
- **departed**: nothing changes in the definition or ledger; departure itself is recorded by **🎭 SonaVida** and the museum, not here.
