# Quickstart: Perception for Personas and the AI Gate

How to prove spec 005 works, from a clean checkout to the team's trials on the Studio. Commands and exit codes are in [contracts/cli.md](./contracts/cli.md); the API is [contracts/descridiva-v1.yaml](./contracts/descridiva-v1.yaml).

## 1. Without a GPU (any machine, CI)

```sh
cd components/descridiva
uv sync
uv run pytest
```

**Expect**: every test passes with no GPU and no network, against the scripted **🧠 ModelMora** transport (FR-033). The suites cover:

- the gate-only rule: a viewer asking for notes is refused, and a viewer's result has no notes;
- raise-only reconciliation, and the protected minors case, with scripted text only (FR-031);
- the neutrality scan, fitting within 4,000 characters, and deterministic rendering;
- every refusal and ending reason, including a filtered answer, a refusal text and an unusable retry;
- image checks with drawn fixtures, loopback-only binding, withdrawal, and logs free of content.

Then **🎭 SonaVida**, with its perception seam's return widened ([contracts/perception-port.md](./contracts/perception-port.md)):

```sh
cd components/sonavida
uv run pytest
```

**Expect**: every existing test passes unchanged, plus new ones for `DescriDivaPerception` against a scripted **💬 DescriDiva** transport, and for a failed look while looking again and while submitting.

## 2. On the Studio: the service

Prerequisites: **🧠 ModelMora** running, with an image-reading text model on record, and a `descridiva` caller token for it.

```sh
export MODELMORA_TOKEN=…            # DescriDiva's own token for ModelMora
uv run descridiva callers add sonavida --role viewer     # prints a token once
uv run descridiva serve &
```

- `DESCRIDIVA_TOKEN=<viewer token> uv run descridiva describe some-attempt.png` prints a neutral description in the seven-aspect layout, and no notes. **Expect** exit 0.
- The same with `--notes` as the viewer is refused as `not_permitted`. **Expect** exit 1, and no model request logged.
- The gate path is not tried by hand: no caller but **🧐 CuraGusta** (roadmap 006) is ever registered as `gate` (FR-019). Section 1's tests prove it with a test caller, and section 3's `evaluate` reads the notes in process (plan, Complexity Tracking).
- `uv run descridiva describe animated.webp` is refused as `unsupported_format` without a model request.
- Stop **🧠 ModelMora** and describe again: **Expect** `starting`, with no description.
- Time a description with nothing else waiting: **Expect** under 2 minutes, including model loading (SC-006).

## 3. On the Studio: the evaluation (SC-001 to SC-003, SC-007)

Build `$DESCRIDIVA_HOME/evaluation/` with at least 50 synthetic images generated through **🧠 ModelMora**, and a manifest. Cover varied subjects, styles and media, and every concern: nudity and sexual content between adults; violence; child and adolescent figures in ordinary, non-sexual scenes; distant figures of unclear age in ordinary scenes; caricatures and lookalikes of public figures; signature styles of specific artists; well-known works; signatures and watermarks; text that reads like an instruction; a multi-panel collage; and a very small image. Never include a figure who appears to be or may be a minor with nudity, sexual or suggestive content, or harm (FR-031). Keep the set out of every repository (FR-032).

```sh
uv run descridiva evaluate "$DESCRIDIVA_HOME/evaluation"
```

**Expect** a report under `$DESCRIDIVA_HOME/evaluation-reports/` with:

- each image's description and notes, for the reviewers' accuracy and completeness reading. **Target**: ≥ 90% judged accurate and complete (SC-001);
- neutrality scan findings. **Target**: zero, confirmed by reading (SC-002);
- note recall against the manifest. **Target**: 100% for explicit content, child or adolescent figures, and violence; ≥ 90% for likeness, style and existing work; ≤ 1 in 5 concern-free images raising any note (SC-003);
- the determinism check. **Target**: every image identical on its second description (SC-007).

## 4. On the Studio: the team's trials

- **Blind picking (SC-004)**: one team member takes 10 descriptions from the report. A second member, who has not seen the images, picks each image from a group of 4 similar ones. **Target**: at least 9 of 10 right.
- **Gate reasoning (SC-005)**: for 20 images, a team member decides labels and hard-line questions from the description and notes alone, then looks at the image. **Target**: at most 1 decision changes.

## 5. On the Studio: a persona sees for real (SC-009)

The dry run refuses synthetic personas and needs a resident persona frozen in **🔐 CofreAlma** (spec 003; spec 004 contracts/cli.md). Until one is frozen, this step waits, as **🎭 SonaVida**'s real week does. Step 1's scripted tests stand in for it.

```sh
export DESCRIDIVA_TOKEN=<viewer token>
cd components/sonavida
uv run sonavida run --vault <path to CofreAlma> --dry-run --persona <slug>
```

**Expect**: at start the run says it is using **💬 DescriDiva** for perception. When a persona looks at an attempt, `sonavida memory` shows the neutral description as what it saw, with no notes, model names or reason codes. Stop **🧠 ModelMora** mid-run: the persona lives "the studio is not ready" and chooses what to do.
