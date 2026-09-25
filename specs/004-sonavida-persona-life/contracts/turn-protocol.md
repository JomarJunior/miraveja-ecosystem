# Turn Protocol, version 1

What **🎭 SonaVida** gives the persona's model at each turn, and what it accepts back (R-1, R-10).

## Given

1. **Who I am**: the persona's `self` record (voice, taste, tendencies, cares, craft, self-image, lore) and that it is an AI whose past is a story it carries.
2. **Where I am**: the date and time, its presence, how long since its last turn, and, if it was away because the Studio was off, for how long (FR-006, FR-009).
3. **What I am doing**: the piece in progress with its attempts and what it saw in them, or nothing.
4. **What I remember**: the recalled entries (R-2), each as its own event in time order. Never a count, total, average, rank or comparison (FR-027).
5. **What I could do now**: the proposals (below), each with a tendency hint where the persona's own habits suggest one.
6. **How to answer**: the reply schema.

Nothing given names a model, a queue, an error code or any Studio internal (FR-031).

## Actions (the closed set)

Every action valid in the current state is always proposed. The hints only order and annotate (R-1).

| Action | Valid when | Details the reply carries |
|---|---|---|
| `set-presence` | always | `presence`: `in-the-studio` \| `resting` \| `away` |
| `form-intention` | in the studio, nothing in progress | `intention` (what and why) |
| `make-attempt` | in the studio, a piece in progress | `ask` (what to make, in its words) |
| `look-again` | a piece in progress with attempts | `attempt` |
| `rework` | a piece in progress with attempts | `attempt`, `ask` |
| `finish` | a piece in progress with a kept attempt | `attempt`, `title`, `statement` |
| `abandon` | a piece in progress | — |
| `submit` | a finished piece not yet decided | `piece`, optional `suggestedLabels` (`explicit`, `violent`) |
| `keep` | a finished piece not yet decided | `piece` |
| `continue-unfinished` | an interrupted piece exists | `piece` |
| `do-nothing` | always | — |
| `leave-the-museum` | always | — (takes effect on the second consecutive choice, R-11) |

## Reply (JSON)

```json
{
  "action": "form-intention",
  "details": { "intention": "a harbor at the moment the fog lifts" },
  "reason": "The fog this morning reminded me of Brinewick.",
  "remember": { "importance": 3 },
  "nextTurnIn": "PT2H"
}
```

| Field | Rules |
|---|---|
| `action` | One of the actions valid in the state. |
| `details` | Exactly the fields the action needs; no others. |
| `reason` | Required, in the persona's own words (FR-007, FR-010, FR-017). |
| `remember.importance` | 1–5, the persona's own sense of how much this matters. |
| `nextTurnIn` | ISO 8601 duration, any length the persona wants (R-6). |

An invalid reply is retried once with the validation message; a second invalid reply becomes a `lost-thread` entry and the persona rests until a default next turn of one hour (R-10). The retry and the default are never used to change what the persona chose.
