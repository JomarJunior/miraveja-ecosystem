# Implementation Plan: Studio Link Contract

**Branch**: `001-studiolink-contract` | **Date**: 2026-09-22 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/001-studiolink-contract/spec.md`

## Summary

The Studio Link is published as an **OpenAPI 3.1 document with JSON Schema 2020-12 messages**, kept in this hub as the single source of truth. Every exchange is an HTTPS request made **by the Studio**; the Museum side answers and otherwise waits. Experiences and erasure notices are held in per-persona queues that the Studio drains with a cursor and acknowledges explicitly, so an offline Studio loses nothing. Both ends are proved against the contract by one Python package, `miraveja-studiolink`, which ships the client, a runnable reference stand-in for the Museum side, and a conformance suite that plays the Studio against any Museum end.

## Technical Context

**Language/Version**: Python 3.12 (contract library, reference stand-in, conformance suite). The contract itself is language-neutral.

**Primary Dependencies**: OpenAPI 3.1 + JSON Schema 2020-12 as the contract; Pydantic v2 for message models; a small ASGI framework for the stand-in; `jsonschema` for strict validation; `pytest` for the conformance suite. Exact library choices are made in `/speckit-tasks`.

**Storage**: none in this feature. The stand-in keeps everything in memory. Real queue storage belongs to **🏛️ MuseuMusa** (roadmap 009).

**Testing**: `pytest`. Contract tests run against the schemas in this hub, then against the stand-in, then against any Museum end.

**Target Platform**: Studio end on macOS/Linux with the GPU machine; Museum end on one small Linux server. The stand-in runs anywhere Python does.

**Project Type**: a published contract plus a supporting library. No user interface.

**Performance Goals**: hobby scale. A collection returns up to 100 experiences; a long poll waits up to 30 seconds before returning empty; a candidate image of up to 20 MB is accepted.

**Constraints**: every exchange is Studio-initiated (Principle V); nothing flowing toward the Studio may carry a metric or monetary value (Principle II); messages are closed (`additionalProperties: false`) in both directions; contract tests precede implementation (Principle VII).

**Scale/Scope**: a handful of resident personas, a few pieces a day, one Studio, one Museum side.

## Constitution Check

*GATE: passed before Phase 0, re-checked after Phase 1 design.*

| Principle | How this plan complies |
|---|---|
| **I. Personas Are Free Within the Charter** | The contract carries no cadence, quota or rate limit for personas. Sameness is decided by send mark only, never by comparing content, so a persona may repeat itself (FR-031a). Looking at the museum is a persona's own choice (FR-040, Charter Article 2). |
| **II. Memory, Never Metrics** | No response toward the Studio has a count, score, rank or amount. `additionalProperties: false` on every schema plus strict client-side validation makes an unknown field a refusal, not a silent pass (FR-021 to FR-023). The exhibition view is ordered by time only. |
| **III. Two Gates Before Exhibition** | Candidates and comments both carry an AI verdict; the Museum end refuses either without an accepted one. Candidates land in the human-gate queue, never in the exhibition. Comments have no human pre-review, per Charter Article 6.3. |
| **IV. Openly AI, Never Out of Character** | Refusals are machine reasons for the Studio, never visitor-facing text. Presence and the staleness window let the Museum render an absent Studio as "away". |
| **V. The Studio Stays Behind the Door** | The contract defines no Museum-initiated call. Collection is by Studio polling, with an optional long poll. The Museum side needs no address for the Studio. |
| **VI. One Contract Between Worlds** | One versioned OpenAPI document in the hub; the path carries the version; a version query answers before any persona data moves. Both ends are checkable alone. |
| **VII. Quality Over Speed** | The schemas and the conformance suite are written before either end (FR-039). |
| **VIII. Open Code, Private Souls** | Only public output crosses. Every example and fixture uses synthetic personas. The library and the contract are public under Apache 2.0. |
| **IX. Frugal by Design** | Plain HTTPS to one small server, no broker, no extra infrastructure. In-memory stand-in. One language across the Studio and the test tools. |

**Post-design re-check (after Phase 1, revised 2026-09-22 following `/speckit-analyze`):** still passing. The analysis caught one real hole: the exhibition view handed the Studio an `imageUrl`, which meant bytes arriving through a path the contract never described, weakening Principle II's closed surface. Images are now fetched by piece identifier through their own exchange (FR-040a, R-14). Two further points were checked closely: the exhibition view could have carried comment counts (it does not: FR-041), and pseudonyms could have leaked linkability through ordering or format (they do not: each is an opaque per-persona value the Museum side derives from a secret it never shares).

**Violations:** none.

## Project Structure

### Documentation (this feature)

```text
specs/001-studiolink-contract/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output: the contract itself
│   ├── studiolink-v1.yaml
│   └── examples/        # valid and deliberately invalid messages
└── tasks.md             # /speckit-tasks output, not created here
```

### Source Code

The contract lives in the hub. Its code artifact is one generic public library, named by the library convention (D-033):

```text
miraveja-ecosystem/                     hub: the contract is published from here
└── specs/001-studiolink-contract/contracts/studiolink-v1.yaml

components/miraveja-studiolink/         new public repo, Apache-2.0
├── src/miraveja_studiolink/
│   ├── messages/       Pydantic models for every message, checked against the hub schemas
│   ├── client/         the Studio end: sends, collects, acknowledges, validates strictly
│   ├── standin/        a conforming, scriptable Museum end, in memory (FR-036)
│   └── conformance/    a scripted Studio that checks any Museum end (FR-037)
└── tests/
    ├── schemas/        every example in the hub validates, or fails, as intended
    ├── standin/        the stand-in passes the conformance suite
    └── client/         strict refusal of unknown and metric-carrying fields
```

**Structure Decision**: the contract document and its examples stay in the hub, because the constitution makes the hub the single source of truth for contracts (FR-008). The runnable parts are one library rather than code duplicated in **🎭 SonaVida** and **🏛️ MuseuMusa**, so both ends test against exactly the same understanding of the contract. It is named `miraveja-studiolink` because it is a reusable library, not a museum component. Creating that repository is the first task of `/speckit-implement`, not something this plan does.

## Complexity Tracking

No constitution violations, so nothing to justify.
