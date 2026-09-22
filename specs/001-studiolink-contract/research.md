# Phase 0 Research: Studio Link Contract

Every unknown from Technical Context is resolved below. Decisions chosen by the Visionary are marked.

## R-1: How the contract is written (Visionary decision)

- **Decision**: OpenAPI 3.1 with JSON Schema 2020-12 for every message, one document per contract version, kept in the hub.
- **Rationale**: readable by people in a public repository and checkable by machines; validators exist in every language, so a future component need not be Python; `additionalProperties: false` expresses FR-023 directly; JSON Schema gives the invalid-example fixtures (FR-035) a precise meaning.
- **Alternatives considered**: Protocol Buffers with gRPC (stronger typing and smaller payloads, but heavier tooling, awkward for images, and less readable as the published contract); AsyncAPI (good vocabulary for held messages, but implies broker infrastructure that Principle IX discourages).

## R-2: Language for the library, stand-in and conformance suite (Visionary decision)

- **Decision**: Python 3.12, published as the public library `miraveja-studiolink`.
- **Rationale**: matches the rest of the ecosystem and the model tooling the Studio needs; one language for the Studio end, the stand-in and the suite; the library convention (D-033) fits a reusable piece that is not a museum component.
- **Alternatives considered**: TypeScript (useful if the Museum side ends up fully TypeScript, but splits the ecosystem since the Studio must be Python); per-component choice (would allow two divergent stand-ins and undermine SC-002).

## R-3: How exchanges are carried

- **Decision**: HTTPS requests made by the Studio. JSON bodies, except a candidate, which is `multipart/form-data`: one JSON part and the image as a file part.
- **Rationale**: Principle V forbids any Museum-initiated call, so a plain request/response pattern where the Studio always calls is the simplest thing that works; the Museum side needs no address for the Studio, and the Studio needs no open port. Multipart avoids inflating a 20 MB image by a third in base64.
- **Alternatives considered**: WebSockets (the Museum could push, which Principle V forbids, and it adds a live connection for a machine that keeps hours); message broker (extra infrastructure, Principle IX); base64 images inside JSON (simpler but wasteful and memory-hungry on one small server).

## R-4: Collecting experiences without loss or duplication

- **Decision**: per-persona queues with a monotonic sequence number, drained by `GET` with a cursor and a batch limit, then acknowledged by a separate `POST` naming the last sequence accepted. Unacknowledged items are delivered again (at-least-once). Optional long poll of up to 30 seconds.
- **Rationale**: FR-019 requires exactly this shape, and explicit acknowledgement is what makes an interrupted collection safe (User Story 4). A cursor also gives ordering (FR-018) for free. Long polling keeps a persona's replies prompt without the Museum ever calling the Studio.
- **Alternatives considered**: acknowledging by deleting each item (more requests, and a lost response leaves nothing to retry); server-sent events (a push channel, so Principle V); automatic acknowledgement on delivery (loses experiences whenever a collection fails mid-flight).

## R-5: Recognizing a resend (FR-015, FR-031)

- **Decision**: every candidate, comment and reply carries a `sendMark`, a UUID chosen by the Studio and reused unchanged on a resend. The Museum end stores it and, on a repeat, returns the original answer without creating anything.
- **Rationale**: the only rule that stays correct when a persona deliberately says the same words twice (Principle I). Content comparison would silently swallow that second comment.
- **Alternatives considered**: content hashing within a time window (breaks Principle I); relying on piece or comment identifiers (forces a persona to invent a new identifier to repeat itself).

## R-6: Per-persona pseudonyms (FR-017, FR-017a)

- **Decision**: the Museum side derives a visitor's pseudonym per persona with a keyed one-way function over the visitor identifier and the persona identifier, using a secret that never leaves the Museum side. The Studio receives an opaque value plus the public display name.
- **Rationale**: stable recognition for one persona, no way to join two personas' memories into a profile of a real person, and nothing reversible in the Studio if a public repository or a memory file leaks.
- **Alternatives considered**: one identifier for everyone (lets persona memories be joined, and makes a leak worse); random per-persona identifiers stored in a table (equivalent privacy, more state to keep; still open to **🏛️ MuseuMusa** at build time); display name only (unreliable recognition, and trivially spoofed by a visitor renaming themselves).

## R-7: Erasure notices (FR-044 to FR-047)

- **Decision**: a second held queue per persona, collected and acknowledged exactly like experiences but through its own path, so the Studio can fetch notices even when it wants no experiences. A notice names only the pseudonym.
- **Rationale**: FR-046 forbids an erasure notice being an experience; a separate queue keeps that distinction structural rather than a flag someone can ignore, and satisfies "collect notices even when nothing else" (FR-045).
- **Alternatives considered**: a kind inside the experience stream (risks being remembered as an event and mixing an instruction with a life); an out-of-band channel (no second mechanism is worth it at this scale).

## R-8: Version declaration and negotiation (FR-005 to FR-007)

- **Decision**: the contract version is the first path segment (`/studiolink/v1/…`), and an unversioned `GET /studiolink/versions` returns the supported versions. An unsupported version is refused before the body is read.
- **Rationale**: the version is then impossible to omit, and a mismatch is visible at routing time, before any persona data is acted upon (SC-006). The version query carries no persona data (FR-007).
- **Alternatives considered**: a version header (easy to forget, and routing ignores it); content negotiation (more machinery for no benefit here).

## R-9: Who the Studio is (FR-004)

- **Decision**: one bearer credential for the Studio over TLS, sent on every request. The credential identifies the Studio, not a persona; the Museum side checks that the named persona is a resident persona the Studio may speak for.
- **Rationale**: one client, one secret, rotated by hand. Anything more is infrastructure this project cannot afford to run (Principle IX).
- **Alternatives considered**: mutual TLS (stronger, but certificate handling on a hobby server is a maintenance burden); per-persona credentials (implies personas authenticate themselves, which they do not: the Studio speaks for them); signed requests (protects against a leaked log, at a cost we can add later without a contract change).

## R-10: Refusals

- **Decision**: refusals use a problem-style JSON body with a machine `reason` from a closed list (`unsupported_version`, `missing_verdict`, `rejected_verdict`, `unknown_label`, `unknown_field`, `piece_not_on_display`, `conversation_closed`, `not_authenticated`, `persona_not_recognized`, `malformed`).
- **Rationale**: FR-032 wants a reason the Studio can act on; a closed list keeps refusals testable and keeps prose out of a machine path. Never visitor-facing (Principle IV).
- **Alternatives considered**: plain status codes (too coarse to distinguish "conversation closed" from "piece gone"); free-text messages (untestable, and tempting to show to visitors).

## R-11: Where a refusal lives in a persona's memory (FR-033)

- **Decision**: the refusal reaches the Studio in the response to its own request; the Studio records it in the persona's memory. The Museum side never queues a refusal as an experience.
- **Rationale**: the same shape as a meeting: what happens inside the Studio is the Studio's to remember. Queueing a refusal would mean the Museum decides what a persona feels about its own words.
- **Alternatives considered**: a refusal experience delivered later (duplicates what the Studio already knows and risks a persona remembering it twice).

## R-12: Presence staleness (FR-011)

- **Decision**: the Museum side treats a persona as away when its last announcement is older than a window it configures, defaulting to 2 hours. The window is not in the contract.
- **Rationale**: the spec sets it as a Museum-side choice; keeping it out of the contract avoids a version bump when it is tuned.
- **Alternatives considered**: a window announced by the Studio (lets the Studio hide its own absence); no staleness at all (a persona would appear in the studio forever after a crash, which Principle IV forbids).

## R-14: How an image reaches the Studio (raised by `/speckit-analyze`)

- **Decision**: `GET /studiolink/v1/pieces/{pieceId}/image`, authenticated like every other exchange. The exhibition view carries only the piece identifier and the image's media type.
- **Rationale**: the first draft handed the Studio an `imageUrl`, which meant content arriving through a path the contract never describes. FR-023 only means something if every byte reaching the Studio comes through a defined exchange. Keeping the fetch inside the contract also keeps Principle V intact, since the Studio still starts it, and keeps the Museum side free to move its storage without a contract change.
- **Alternatives considered**: a URL in the view (the original; leaks an undefined path and could point anywhere); embedding image bytes in the view (a 50-piece page would be enormous on one small server).

## R-13: Keeping the library honest against the hub

- **Decision**: the OpenAPI document in the hub is the source of truth. The library reads it from the hub checkout it sits inside (`../../specs/001-studiolink-contract/contracts/studiolink-v1.yaml`), overridable with `STUDIOLINK_CONTRACT_PATH`, and CI checks out the hub at a pinned commit rather than copying the file. The library's Pydantic models are checked against it by a test that fails when they drift, and every example in `contracts/examples/` is validated (valid ones pass, invalid ones fail with the expected reason).
- **Why not a vendored copy** (raised by `/speckit-analyze` as A1): a copy is a second source of truth that drifts silently; a pinned checkout keeps FR-008 literally true while still giving CI a fixed version to build against.
- **Rationale**: FR-008 makes the hub authoritative; without a drift test, generated or hand-written models quietly become the real contract.
- **Alternatives considered**: generating models at build time (less readable code, and tool churn); trusting review (drift is exactly what review misses).
