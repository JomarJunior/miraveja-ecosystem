# Data Model: A Persona's Life in the Studio

Everything here is **🎭 SonaVida**'s own data, on the Studio, one directory per persona:
`$SONAVIDA_HOME/personas/<persona-id>/` holding `memory.sqlite` and `lock`. Nothing in it
ever leaves the Studio (FR-034).

## `self` (one row, written once at birth)

| Field | Type | Rules |
|---|---|---|
| `persona_id` | UUID | From the definition; the Studio Link `personaId`. |
| `public_name` | text | From the definition. |
| `self_knowledge` | JSON | Identity, taste, voice, tendencies, cares, craft, self-image and lore, copied from the definition at birth (R-3). Never rewritten. |
| `born_at` | timestamp | When it came alive here. |
| `departed_at` | timestamp, nullable | Set once, on leaving (FR-040). |

## `entries` (append-only)

| Field | Type | Rules |
|---|---|---|
| `id` | integer | Monotonic; defines time order. |
| `at` | timestamp | Persona time (the `Clock`). |
| `kind` | enum | See below. |
| `text` | text | In the persona's own words where it is the persona's decision; neutral otherwise. Visitors appear only as `⟨v:pseudonym⟩` tokens (R-8). |
| `reason` | text, nullable | The persona's reason, for its own decisions (FR-007, FR-010, FR-017). |
| `importance` | integer 1–5 | Chosen by the persona when it forms the memory; used only for recall ordering, never shown as a score. |
| `piece_id` | UUID, nullable | The piece the entry concerns. |
| `visitor_pseudonym` | text, nullable | Cleared by erasure (FR-032). |
| `visitor_name` | text, nullable | Cleared by erasure. |
| `source_sequence` | integer, nullable | Studio Link sequence of an experience, for once-only delivery (FR-022). |

Rows are never deleted (FR-026). The only update ever made to an existing row is erasure, which clears the two visitor columns and replaces tokens (R-8). An FTS5 table `entries_fts(text, reason)` mirrors `entries` for recall.

**Kinds**: `seed` · `self-aware` (it is an AI, FR-002) · `presence` · `time-away` · `intention` · `attempt` · `attempt-seen` (perception) · `finished` · `abandoned` · `kept` · `submitted` · `verdict` · `experience` · `studio-not-ready` · `interrupted` · `attempt-failed` · `lost-thread` · `thinking-of-leaving` · `departed` · `chose-nothing`.

## `pieces`

| Field | Type | Rules |
|---|---|---|
| `id` | UUID | Also the candidate's piece identifier on the Studio Link. |
| `intention_entry` | integer | The `intention` entry it grew from. |
| `state` | enum | `in-progress` · `finished` · `abandoned` · `kept` · `submitted` · `accepted` · `rejected` · `exhibited` · `declined` · `taken-down`. |
| `title`, `statement` | text, nullable | Set when finished; the persona's words (FR-014). |
| `suggested_labels` | set of `explicit` \| `violent` | The persona's own suggestion (FR-042). |
| `labels` | set, nullable | The gate's decision; the persona cannot change it. |
| `image_path` | path | The chosen attempt's image, in the persona directory. |

State moves only forward along: `in-progress → finished → kept | submitted`, `in-progress → abandoned`, `submitted → accepted | rejected`, `accepted → exhibited | declined`, `exhibited → taken-down`. Kept, abandoned and rejected pieces never leave the Studio (SC-004). Nothing is altered or withdrawn by an erasure (FR-033).

## `attempts`

| Field | Type | Rules |
|---|---|---|
| `id`, `piece_id` | — | |
| `asked` | text | What the persona asked for, in its words. |
| `image_path` | path, nullable | Result, if any. |
| `seen` | text, nullable | The perception description. |
| `outcome` | enum | `kept-as-final` · `discarded` · `reworked` · `did-not-come-out` · `interrupted`. |

## `inbox` (Studio Link bookkeeping, no persona content)

| Field | Rules |
|---|---|
| `experiences_acknowledged_through` | Last sequence acknowledged, so a restart loses and repeats nothing (SC-005). |

## Turn state (in memory only, rebuilt from `entries` on start)

| Field | Meaning |
|---|---|
| `presence` | `in-the-studio` · `resting` · `away`. |
| `working_on` | Current `in-progress` piece, if any. |
| `waiting_for` | A pending **🧠 ModelMora** request, if any. |
| `next_turn_at` | Chosen by the persona at its last turn. |
| `leaving_pending` | True after one `thinking-of-leaving` (R-11). |

## Presence transitions

Any state → any state, chosen by the persona (FR-005). Imposed transitions are only: → `away` on orderly shutdown (FR-008), and "was away" on return (FR-009). Each chosen change is announced and remembered (FR-007).
