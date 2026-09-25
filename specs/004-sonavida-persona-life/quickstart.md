# Quickstart: A Persona's Life in the Studio

Runnable scenarios that prove spec 004 works. They use synthetic personas and stand-ins only; no resident persona is involved (FR-037). Interfaces: [contracts/cli.md](./contracts/cli.md), [contracts/turn-protocol.md](./contracts/turn-protocol.md), [contracts/ports.md](./contracts/ports.md).

## Prerequisites

- Python 3.12 and `uv`; `components/sonavida/` installed with `uv sync --all-groups`.
- A scratch `SONAVIDA_HOME`: `export SONAVIDA_HOME=$(mktemp -d)`.
- A scratch vault of synthetic personas, built by the test fixtures from the spec 003 hub examples.

## 1. A persona comes alive and keeps its hours (US1, SC-003)

```bash
uv run sonavida run --simulate 7 --seed 1 --standins
uv run sonavida memory pellam-quist | head -40
```

Expected: one birth; seed memories and shared pasts first, told in the first person; every presence change announced on the reference stand-in and explained in memory; the gap when the simulated Studio was off remembered as time away.

## 2. It makes pieces and names them (US2)

`uv run sonavida pieces pellam-quist`. Expected: each finished piece has an intention, attempts with what the persona saw, a title and a statement in its voice.

## 3. It decides what to show (US3, SC-004)

`uv run pytest tests/integration/test_showing_work.py`. Expected: only submitted pieces reach the gate stand-in; only accepted ones reach the Studio Link stand-in, with the gate's labels; rejections return as memories with feedback.

## 4. It remembers what happened, with no numbers (US4, SC-005, SC-006)

`uv run pytest tests/integration/test_experiences.py tests/privacy/test_no_metrics.py tests/privacy/test_erasure.py`. Expected: each scripted experience remembered once, in order, across a restart; no count, total or rank in any prompt or memory; after an erasure the pseudonym and name are gone from the file (checked on the raw bytes after `VACUUM`), and the memories remain.

## 5. The Studio's limits are part of its life (US6, SC-007)

`uv run pytest tests/integration/test_studio_limits.py`. Expected: with the models stand-in scripted *busy*, *starting* and *stopping*, no intention is lost or reduced, and memory has no model names, codes or queue positions.

## 6. Several personas, each alone (FR-041, SC-010)

```bash
uv run sonavida run --simulate 7 --seed 2 --standins --persona pellam-quist --persona ivo-marrowfield --persona test-persona-nine
```

Expected: three separate memory files; no entry of one in another; every intention carried out, set aside or waiting.

## 7. The team reads, never writes (US5, SC-002)

A team member who did not watch scenario 1 answers the fixed ten questions in `tests/fixtures/reading-questions.md` from `sonavida memory` alone. Expected: at least 9 of 10 correct in under 30 minutes. `uv run pytest tests/privacy/test_read_only.py` proves no write path exists.

## 8. Leaving (FR-040)

`uv run pytest tests/integration/test_leaving.py`. Expected: one `leave-the-museum` choice is remembered as thinking of leaving; a second consecutive one records the departure, announces away once, and `sonavida run` never starts that persona again.
