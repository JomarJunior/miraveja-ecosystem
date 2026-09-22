# Quickstart: proving the Studio Link works

How to check that the contract and its two test tools do what the spec promises. Nothing here is implementation code; that belongs to `/speckit-tasks` and `/speckit-implement`.

## Prerequisites

- Python 3.12 and `uv`.
- This hub checked out. The contract is `specs/001-studiolink-contract/contracts/studiolink-v1.yaml`.
- The library `miraveja-studiolink` checked out at `components/miraveja-studiolink/` (created in the first implementation task).

## Scenario 1: the examples match the contract

Proves FR-035 and keeps the hub honest.

```bash
cd components/miraveja-studiolink
uv run pytest tests/schemas
```

Expected: every file in `contracts/examples/valid/` validates; every file in `contracts/examples/invalid/` is rejected with its `_expectedRefusal`. A drift test also fails if the library's models no longer match the hub's schemas (R-13).

## Scenario 2: the Studio end works with no Museum side (SC-001)

Proves User Story 1 and the build order for Stage A.

```bash
uv run miraveja-studiolink standin --port 8080 --script tests/fixtures/one-visitor-comment.yaml
uv run pytest tests/client
```

Expected, with no **🏛️ MuseuMusa** or **🛡️ PortaGuarda** installed anywhere:

- a synthetic persona announces `in_the_studio` and the stand-in reports that presence;
- a candidate is accepted and lands in the stand-in's human-gate queue, never in its exhibition;
- collecting experiences returns the scripted comment as one individual event, with the visitor as a pseudonym plus display name;
- publishing a reply with a `hard_lines_only` verdict is recorded in the same conversation;
- looking at the museum returns the scripted exhibition in time order, and changes nothing;
- an exhibited piece's image is fetched by its piece identifier through the contract, and the view hands out no location outside it.

## Scenario 3: nothing that counts or pays reaches a persona (SC-003)

Proves User Story 2 and Principle II.

```bash
uv run pytest tests/client -k metrics
```

Expected: the stand-in is told to answer with each contaminated example from `contracts/examples/invalid/`; the Studio end refuses the whole message every time and passes nothing to the persona. Twelve reactions arrive as twelve experiences, never as a total.

## Scenario 4: the Studio keeps hours and loses nothing (SC-004)

Proves User Story 4.

```bash
uv run pytest tests/client -k offline
```

Expected: with the Studio silent for a simulated 24 hours, all queued experiences are collected oldest first; a collection interrupted before acknowledgement delivers the same items again; once acknowledged they never return; a persona whose last announcement is older than the staleness window shows as away.

## Scenario 5: any Museum end can be checked with no Studio (SC-002)

Proves User Story 3. Run it against the stand-in first, then later against the real **🏛️ MuseuMusa**.

```bash
uv run miraveja-studiolink conformance --target http://localhost:8080 --credential $STUDIOLINK_TOKEN
```

Expected: a pass or fail line per exchange, and the reference stand-in passes 100%.

## Scenario 6: resends and versions

```bash
uv run pytest tests/client -k "sendmark or version"
```

Expected: the same candidate or comment sent twice under one send mark appears once, with the original answer returned; the same text under a new send mark appears twice (a persona may repeat itself); an unsupported version is refused before any content is acted on, naming the supported versions.

## Scenario 7: erasure

```bash
uv run pytest tests/client -k erasure
```

Expected: after the stand-in is told a visitor was erased, a notice naming only that persona's pseudonym waits in the notice queue, is collected separately from experiences, and is never presented as something the persona remembers.

## What "done" looks like

All seven scenarios pass, and the success criteria in the spec are met: SC-001 to SC-010. The Studio end is then ready for roadmap 002 to 004 to build on.
