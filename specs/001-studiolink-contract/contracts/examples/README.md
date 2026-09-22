# Studio Link examples

Message examples for contract version 1 (FR-035). Every persona and visitor here is synthetic; no real persona definition or name appears (Principle VIII).

Each file carries three keys that are **not part of the contract** and must be stripped before validating:

- `_schema`: which schema in `../studiolink-v1.yaml` the message is an instance of.
- `_expectedRefusal` (invalid only): the refusal reason the Museum or Studio end must answer with.
- `_why` (invalid only): the rule the message breaks.

## Valid

| File | Schema |
|---|---|
| `valid/presence-announcement.json` | PresenceAnnouncement |
| `valid/candidate.json` | Candidate (the JSON part of the multipart request) |
| `valid/persona-comment.json` | PersonaComment |
| `valid/experience-batch.json` | ExperienceBatch: one comment, one reaction, one gate outcome |
| `valid/erasure-notice-batch.json` | ErasureNoticeBatch |
| `valid/refusal-conversation-closed.json` | Refusal |

## Invalid, and the rule each breaks

| File | Expected refusal | Rule |
|---|---|---|
| `invalid/experience-batch-with-count.json` | `unknown_field` | No count may flow toward the Studio (FR-021, FR-023) |
| `invalid/verdict-with-score.json` | `unknown_field` | A verdict carries no score (Principle II) |
| `invalid/comment-with-tone-judgment.json` | `unknown_field` | No judgment of tone or conduct (FR-028, Charter Article 7) |
| `invalid/experience-meeting-kind.json` | `unknown_field` | Meetings live only in the Studio (FR-042) |
| `invalid/candidate-unknown-label.json` | `unknown_label` | Labels are a closed set (FR-013) |
| `invalid/comment-missing-verdict.json` | `missing_verdict` | Comments carry the AI gate's verdict (FR-027) |
| `invalid/presence-unknown-state.json` | `malformed` | Presence is one of three states (FR-009) |

Checked on 2026-09-22 against `studiolink-v1.yaml`: all six valid examples validate, all seven invalid ones are rejected.
