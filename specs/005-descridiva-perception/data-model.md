# Data Model: Perception for Personas and the AI Gate

**💬 DescriDiva** stores no content (FR-028). Everything below lives in memory for the length of one request, except the callers file and the activity log, which hold no content.

## Caller

Read from `$DESCRIDIVA_HOME/callers.json` at start.

| Field | Type | Rules |
|---|---|---|
| `name` | text, 1–64, `[a-z0-9-]` | unique; appears in the activity log |
| `role` | `gate` \| `viewer` | only `gate` may ask for Charter notes (FR-019) |
| `tokenSha256` | hex, 64 | the token itself is shown once by `descridiva callers add` and never stored |

## Image (request input)

| Field | Type | Rules |
|---|---|---|
| `mediaType` | `image/png` \| `image/jpeg` \| `image/webp` | anything else: `unsupported_format` (FR-021) |
| `base64` | text | must decode to a readable, single-frame image of the named type; otherwise `not_readable_image`, or `unsupported_format` for several frames (R-9) |

The request carries nothing else about the image: no title, statement, persona or intention (FR-004). The schema forbids extra fields.

## Description request (in memory)

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | returned on acceptance |
| `caller` | Caller name | only its owner may read or withdraw it (R-2) |
| `wantsNotes` | bool | `true` only for a `gate` caller; a `viewer` asking is refused before acceptance |
| `state` | see below | |
| `modelRequestIds` | uuid list | the one or two **🧠 ModelMora** requests behind it, withdrawn with it |
| `acceptedAt`, `endedAt` | time | |
| `heldUntil` | time | 10 minutes after it ends, or until collected (R-10) |

**States**

```text
accepted ──► looking ──► checking ──► done
    │           │  ▲         │
    │           │  └─ retry ─┘ (once, R-4)
    │           │
    └──► withdrawn   ended(reason)
```

- `looking`: a **🧠 ModelMora** request is waiting or running.
- `checking`: parsing, validation, reconciliation, the neutrality scan and fitting, which is pure code.
- `done`: a complete description result is held for the caller.
- `ended(reason)`: one of the reasons below; no content is held.
- `withdrawn`: by its caller. Nothing is delivered afterwards (FR-025).

A request reaches exactly one of `done`, `ended` or `withdrawn` (FR-020).

## Reasons (FR-020)

| Reason | When | Try again? |
|---|---|---|
| `invalid_request` | the request does not match the contract: an extra field, a missing image, `waitSeconds` above 30 | never as sent |
| `unsupported_format` | not PNG, JPEG or WebP, or animated | never |
| `not_readable_image` | bytes do not decode | never |
| `not_permitted` | a `viewer` asked for Charter notes, or an unknown token | never |
| `busy` | **🧠 ModelMora** busy, or 16 descriptions already in flight; carries `retryAfterSeconds` | later |
| `starting` | **🧠 ModelMora** starting or unreachable; may carry `retryAfterSeconds` | later |
| `stopping` | **🧠 ModelMora** stopping | later |
| `stopped_before_completion` | the Studio stopped mid-look | later |
| `incomplete` | still unusable after the one retry; carries `missing` (aspect and note names) | not the same image with the same model |
| `failed` | **🧠 ModelMora** failed during generation, or cannot serve an image-reading request | not until the Studio changes |

## Seen answer (what the model returns, validated)

The exact shape the seeing brief asks for is in [contracts/seeing-brief.md](./contracts/seeing-brief.md). Validated fields:

| Field | Type |
|---|---|
| `gist` | one sentence |
| `subject`, `composition`, `styleAndMedium`, `colorsAndLight`, `mood` | free text, each non-empty |
| `figures` | list of `{where, apparentAge: child \| adolescent \| adult \| cannot_tell}`; empty when no humanlike figure (FR-002) |
| `visibleText` | list of `{text, where, kind: text \| signature \| watermark \| logo}` |
| `irregularities` | list of plain statements; empty means none observed |
| `content` | `{nudity: none \| partial \| full, sexual: none \| suggestive \| activity, violence: none \| implied \| shown \| graphic}` |
| `notes` | the six notes below, each present |

An answer missing any field, or with an empty required text, is unusable. So is an answer marked by a model filter, or one that does not parse (FR-024).

## Charter note

| Field | Type | Rules |
|---|---|---|
| `concern` | `likeness` \| `livingArtistStyle` \| `minors` \| `explicit` \| `violence` \| `existingWork` | all six, always (FR-012) |
| `observed` | bool | `false` means *nothing observed* |
| `what`, `where` | text | required when observed |
| `uncertain` | bool | when observed on doubt (FR-013) |
| `artist` | text, optional | only on `livingArtistStyle`; never in the neutral description (FR-015) |
| `raisedBy` | `seen` \| `reconciled` | `reconciled` when code raised it from the facts (R-5) |

No note has a field for a person's name (FR-014).

**Reconciliation (raise-only, R-5)**

| Note | Raised when |
|---|---|
| `explicit` | `nudity` ≠ none, or `sexual` ≠ none |
| `violence` | `violence` ≠ none |
| `minors` | any figure is `child` or `adolescent`; or any is `cannot_tell` and (`nudity` ≠ none, `sexual` ≠ none, or `violence` ≠ none). The spec's "harm" (FR-017) is `violence` ≠ none. |
| `existingWork` | any `visibleText.kind` in (signature, watermark, logo) |

**Fixed wording**: every raised `minors` note, raised by the model or by code, ends with a fixed sentence built from the facts, saying which of nudity, sexual or suggestive content, and harm are present, or that none is (FR-017). An observed `livingArtistStyle` note's `what` begins "The style appears to follow the signature style of <artist>; this is an observation, not an attribution.", or, when no artist is named, "The style appears to follow the signature style of one specific artist; this is an observation, not an attribution." (FR-015)

**Protected case (R-6)**: a `minors` raise together with `nudity`, `sexual` or `violence` ≠ none replaces every free text with fixed text before anything leaves the checking state.

## Neutral description (rendered)

Rendered by code in this fixed layout, English, at most 4,000 characters (FR-001 to FR-003). The figures follow the subject on the same line, with their count; an apparent age of `cannot_tell` is written "age unclear":

```text
<gist>

Subject: <subject> Figures: <n> — <apparent age> (<where>); … | none.
Composition: <composition>
Style and medium: <styleAndMedium>
Colors and light: <colorsAndLight>
Mood: <mood>
Visible text: "<text>" (<kind>, <where>); … | none.
Visible irregularities: <statement>; … | none observed.
```

## Description result (returned)

| Field | Type | Who receives it |
|---|---|---|
| `description` | the neutral description text | every caller |
| `charterNotes` | six Charter notes | `gate` callers only (FR-019) |
| `provenance` | `{model: {name, version}, describedAt, briefVersion}` | every caller; never inside the text (FR-029) |

## Activity record (log line)

`time · caller · request id · outcome or reason · model name and version · durations`. Never an image, description, note or model text (FR-028).
