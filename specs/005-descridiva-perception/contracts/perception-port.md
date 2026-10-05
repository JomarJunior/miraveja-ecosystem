# **🎭 SonaVida**'s Perception Port

The same port as spec 004's [ports version 1](../../004-sonavida-persona-life/contracts/ports.md), with its return widened so failures reach the persona's life, as spec 004 FR-029, R-7 and that contract's closing line already require (R-11). Everything else in that contract stands.

| | As built in spec 004 | After this spec |
|---|---|---|
| Signature | `describe(image_bytes) -> str` | `describe(image_bytes) -> str \| Busy \| Starting \| Stopping \| Failed \| CannotServe` |
| Real implementation | a seam for roadmap 005 | `DescriDivaPerception`: loopback client to **💬 DescriDiva** as a `viewer`, never asking for Charter notes (FR-019) |
| Stand-ins | `ModelsPerception` (failure became a sentence), `ScriptedPerception` | `ModelsPerception` returns the `Models` outcome on failure instead of a sentence; `ScriptedPerception` can also script outcomes |

## Mapping **💬 DescriDiva**'s reasons

| **💬 DescriDiva** | Port outcome | What the persona lives (`translate.py`) |
|---|---|---|
| done | the description `str` | it sees its work |
| `busy` | `Busy(retry_at)` | the studio is not ready, perhaps around a time of day |
| `starting` | `Starting(retry_at)` | the studio is not ready |
| `stopping`, `stopped_before_completion` | `Stopping()` | the work was interrupted |
| `incomplete`, `failed` | `Failed()` | the attempt could not be seen clearly |
| `unsupported_format`, `not_readable_image`, `not_permitted`, `invalid_request`, `not_found` | `CannotServe()` | the attempt could not be seen clearly; one operator log line, since this means a defect in **🎭 SonaVida** or its setup |

The client polls one description to its end and never re-asks. Asking again is the persona's choice at a later turn (spec 004 FR-029).

## Where a failed look lands

- **How it is remembered**: with spec 004's existing entry kinds only (`studio-not-ready`, `interrupted`, `attempt-failed`), so spec 004's closed list of kinds is unchanged, and always with the piece's id.
- **Looking again at an attempt**: nothing is stored as seen, and the translated situation is remembered.
- **Submitting a piece**: nothing goes to the gate with a disguised description. The piece stays `submitted`, because spec 004's piece states only move forward (spec 004 data-model.md), and the situation is remembered. At the persona's next turn, a `submitted` piece whose latest entry after its `submitted` entry is one of those three kinds is looked at again and brought to the gate, completing the persona's own choice to submit (R-11, decided by the Visionary). A look that fails again is remembered again.

No model name, reason code or number reaches memory (spec 004 FR-031).
