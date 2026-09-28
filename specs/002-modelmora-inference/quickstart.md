# Quickstart: proving ModelMora works

Runnable checks behind the spec's success criteria. Scenarios 1 to 6 need no GPU: they run in test mode with stand-in models (FR-033). Scenario 7 is the one that needs the Studio.

## Prerequisites

- Python 3.12 and `uv`.
- `components/modelmora/` checked out.
- For Scenario 7 only: the Studio machine, its GPU, the `gpu` extra installed (`uv sync --extra gpu`), a `llama-server` binary for the Studio's `.gguf` text model (see `docs/usage.md`), and the Studio's own model collection registered (`modelmora model add`, `docs/usage.md`), with each single-file image checkpoint's pipeline config and tokenizer directory recorded as its `config` companion (`--companion config=<dir>` at `model add`, or `modelmora model add-companion`) so it loads offline -- **🧠 ModelMora** never downloads models on its own.

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

First register the Studio's own collection (its files already sit on the Studio; see `docs/usage.md`), then:

```bash
uv run python -m modelmora.checks.studio_smoke
```

This runs the real `modelmora serve` as its own subprocess and talks to it over loopback HTTP, exactly as a Studio component would -- no in-process shortcut. It needs a `llama-server` binary (`MODELMORA_LLAMA_SERVER_BIN`, `MODELMORA_LLAMA_CUDART_LIB_DIR`) for a `.gguf` text model; see `checks/studio_smoke.py`'s own docstring for the defaults it falls back to.

Expected: a text model and an image model both serve real results; asking for both in turn lets the real GPU capacity decide whether the text model must be evicted to make room -- nothing reduced by hand -- and the caller sees only a longer wait, never a memory error; the text model reloads correctly afterward; a clean shutdown (the same signal an operator's Ctrl+C sends) exits 0 and returns the GPU to its idle baseline.

Run on the Studio (RTX 4090, driver 595.84) against the team's own registered collection: a text request to the GGUF model in ~4.0-4.2s; an image request to an SDXL checkpoint (832x1216, 24 steps) in ~15.8-20.6s, naturally evicting the text model (GPU ~19.2GB while it was resident, down to ~15.2GB with only the image model); a second text request reloading correctly (~8.6-32.3s including the reload); a clean shutdown, exit 0, GPU back to its ~450MiB baseline. Confirmed twice.

## What "done" looks like

Scenarios 1 to 6 pass in CI with no GPU, Scenario 7 passes on the Studio, and SC-001 to SC-010 are met. **🎭 SonaVida** (roadmap 004) can then be built against test mode.
