# Quickstart: A Persona's Life in the Studio

Runnable scenarios that prove spec 004 works. They use synthetic personas and stand-ins only; no resident persona is involved (FR-037). Interfaces: [contracts/cli.md](./contracts/cli.md), [contracts/turn-protocol.md](./contracts/turn-protocol.md), [contracts/ports.md](./contracts/ports.md).

## Prerequisites

- Python 3.12 and `uv`; `components/sonavida/` installed with `uv sync --all-groups`.
- A scratch `SONAVIDA_HOME`: `export SONAVIDA_HOME=$(mktemp -d)`.
- A scratch vault of synthetic personas: a plain directory holding the spec 003 hub examples.

```bash
VAULT=$(mktemp -d)
EX=specs/003-cofrealma-persona-definition/contracts/examples   # in the hub
cp "$EX/pellam-quist.persona.yaml" "$EX/ivo-marrowfield.persona.yaml" "$VAULT/"
```

`memory` and `pieces` name a persona by its public name or its identifier; `--persona` names a definition file by its slug.

## 1. A persona comes alive and keeps its hours (US1, SC-003)

```bash
uv run sonavida run --vault "$VAULT" --simulate 7 --seed 1 --standins --persona pellam-quist
uv run sonavida memory "Pellam Quist" | head -40
```

Expected: one birth; the persona's awareness that it is an AI, its seed memories and its shared pasts first, told in the first person. Without a script, the models stand-in answers every turn with doing nothing, so this run shows birth and memory but no presence changes. `uv run pytest tests/integration/test_hours.py` scripts them: every presence change announced on the reference stand-in and explained in memory, and the gap when the simulated Studio was off remembered as time away.

## 2. It makes pieces and names them (US2)

`uv run pytest tests/integration/test_making.py tests/integration/test_week.py`. Expected: each finished piece has an intention, attempts with what the persona saw, a title and a statement in its voice. An unscripted stand-in run makes nothing, so `uv run sonavida pieces "Pellam Quist"` after scenario 1 lists no pieces; it is the command to read a scripted or real run.

## 3. It decides what to show (US3, SC-004)

`uv run pytest tests/integration/test_showing_work.py`. Expected: only submitted pieces reach the gate stand-in; only accepted ones reach the Studio Link stand-in, with the gate's labels; rejections return as memories with feedback.

## 4. It remembers what happened, with no numbers (US4, SC-005, SC-006)

`uv run pytest tests/integration/test_experiences.py tests/privacy/test_no_metrics.py tests/privacy/test_erasure.py`. Expected: each scripted experience remembered once, in order, across a restart; no count, total or rank in any prompt or memory; after an erasure the pseudonym and name are gone from the file (checked on the raw bytes after `VACUUM`), and the memories remain.

## 5. The Studio's limits are part of its life (US6, SC-007)

`uv run pytest tests/integration/test_studio_limits.py`. Expected: with the models stand-in scripted *busy*, *starting* and *stopping*, no intention is lost or reduced, and memory has no model names, codes or queue positions.

## 6. Several personas, each alone (FR-041, SC-010)

```bash
uv run sonavida run --vault "$VAULT" --simulate 7 --seed 2 --standins --persona pellam-quist --persona ivo-marrowfield
uv run pytest tests/integration/test_several.py
```

Expected: the run leaves two separate memory files, with no entry of one in another. The test adds a third synthetic persona, Sable Quinn, and scripts a week of work for all three: three separate memory files, no entry of one in another, every intention carried out, set aside or waiting.

## 7. The team reads, never writes (US5, SC-002)

A team member who did not watch scenario 1 answers the fixed ten questions in `tests/fixtures/reading-questions.md` from `sonavida memory` alone. Expected: at least 9 of 10 correct in under 30 minutes. `uv run pytest tests/privacy/test_read_only.py` proves no write path exists.

## 8. Leaving (FR-040)

`uv run pytest tests/integration/test_leaving.py`. Expected: one `leave-the-museum` choice is remembered as thinking of leaving; a second consecutive one records the departure, announces away once, and `sonavida run` never starts that persona again.
