# Quickstart: proving ModelMora works

Runnable checks behind the spec's success criteria. Scenarios 1 to 6 need no GPU: they run in test mode with stand-in models (FR-033). Scenario 7 is the one that needs the Studio.

## Prerequisites

- Python 3.12 and `uv`.
- `components/modelmora/` checked out.
- For Scenario 7 only: the Studio machine, its GPU, the `gpu` extra installed (`uv sync --extra gpu`), and a local, downloaded copy of a small open-weight text model and image model (`MODELMORA_SMOKE_TEXT_MODEL_PATH` and `MODELMORA_SMOKE_IMAGE_MODEL_PATH`; see `checks/studio_smoke.py`'s own docstring for the defaults it falls back to).

## Scenario 1: a caller asks for text and an image, knowing nothing about models

Proves User Stories 1 and 2, and SC-001.

```bash
cd components/modelmora
uv run modelmora serve --test-mode --port 8431 &
uv run pytest tests/api -k "text or image"
```

Expected: a test caller acting as **🎭 SonaVida** submits a text request with no model named, receives an acceptance, then a result carrying the model's name and version. Another acting as **🧐 CuraGusta** sends an image with a question and gets text about it. Neither loads or places a model. Naming an unknown model is refused as `unknown_model`, with no substitute used.

## Scenario 2: twenty requests, four callers, one GPU's worth of room

Proves User Story 3, SC-002 and SC-003.

```bash
uv run pytest tests/queue -k burst
```

Expected: every request is answered within a second with a position and an estimate; each later ends in a result, a failure with a reason, or a withdrawal. Nothing is lost, duplicated or left open. With the line full, further requests are refused as `busy` with a retry time, never accepted and dropped.

## Scenario 3: ordering is about models, never about who asked

Proves FR-019 and SC-010.

```bash
uv run pytest tests/queue -k ordering
```

Expected: requests for a resident model may overtake an older request needing a load, but never for longer than the bounded time; after that the older one runs next. Two callers submitting the same model in a known order are served in that order.

## Scenario 4: nothing is quietly made smaller

Proves FR-007 and SC-008.

```bash
uv run pytest tests/queue -k no_degradation
```

Expected: under load the stand-ins record every generation's settings; across the run, zero results used a model, size, length or quality other than what the request asked for or the acceptance named. A request that cannot be served as asked is refused or fails, with a reason.

## Scenario 5: the registry is the license trail

Proves User Story 4, SC-005 and SC-006.

```bash
uv run modelmora model add --name <name> --version <v> --kind text \
  --license <license> --license-source <url> --source <where> --confirm-license
uv run modelmora model list
uv run pytest tests/registry
```

Expected: a complete record becomes servable; an incomplete one is refused and never loaded; a file digest mismatch refuses the load and reports it; a retired model keeps its record with its service dates; every result produced during the test names a model whose record holds a license.

## Scenario 6: persona prompts leave no trace

Proves FR-030 and SC-007.

```bash
uv run pytest tests/privacy
```

Expected: test callers send requests containing a unique marker phrase; afterwards a search of every log, error report and the registry finds the marker zero times. Records of activity hold the caller, model, settings, times and outcome only.

## Scenario 7: on the Studio, with real models

Not part of CI. Run by hand on the Studio machine.

```bash
uv run python -m modelmora.checks.studio_smoke
```

No separate `modelmora serve` process: the check builds its own service in-process, the same way the test suite does, with real `TextRunner`/`ImageRunner` instances wired directly to the two model paths above -- `modelmora serve`'s own registry-to-runner wiring for the CLI is a deliberate seam (see the comment in `cli.py`), not yet closed by any task, so this is how a real model is exercised end to end today.

Expected: a text model and an image model both serve real results; asking for both in turn forces an eviction and the caller sees only a longer wait, never a memory error; availability reads `starting`, then `running`; a request left open when the check stops ends `stopped_before_completion`.

Run on the Studio (RTX 4090) with a small open text model (text, Apache-2.0) and a small open image model (image, an open model licence): text in ~3.8s, image (384x384, 20 steps) in ~0.8s, the text model correctly evicted to fit the image model under a reduced capacity, and a clean exit 0, twice in a row.

## What "done" looks like

Scenarios 1 to 6 pass in CI with no GPU, Scenario 7 passes on the Studio, and SC-001 to SC-010 are met. **🎭 SonaVida** (roadmap 004) can then be built against test mode.
