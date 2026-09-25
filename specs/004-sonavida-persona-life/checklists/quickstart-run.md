# Quickstart Run: A Persona's Life in the Studio (T051)

Every scenario in `../quickstart.md` was run against the built `sonavida` CLI and
test suite, except scenario 7's human reading trial (that needs a person and is
T052's job). All personas involved are synthetic (FR-037); none of these commands
touched **🔐 CofreAlma**.

## Setup used

```bash
cd components/sonavida   # JomarJunior/sonavida, branch main
uv sync --all-groups
export MIRAVEJA_HUB_PATH=/home/user/miraveja-ecosystem
```

Quickstart's prerequisites list `export SONAVIDA_HOME=$(mktemp -d)` and "a scratch
vault of synthetic personas, built by the test fixtures from the spec 003 hub
examples", but does not give a shell recipe for building that vault outside pytest.
A scratch vault was built by hand for these runs by copying the spec 003 hub
examples into a plain directory of `*.persona.yaml` files (that is all
`sonavida run --vault` requires; the `.cofrealma` marker file the test fixtures
also write is not read on this path):

```bash
VAULT=$(mktemp -d)
EX=/home/user/miraveja-ecosystem/specs/003-cofrealma-persona-definition/contracts/examples
cp "$EX/pellam-quist.persona.yaml" "$VAULT/"
cp "$EX/ivo-marrowfield.persona.yaml" "$VAULT/"
```

## 1. A persona comes alive and keeps its hours (US1, SC-003)

Commands run:

```bash
export SONAVIDA_HOME=$(mktemp -d)
uv run sonavida run --vault "$VAULT" --simulate 7 --seed 1 --standins
uv run sonavida memory "Pellam Quist" | head -40
```

Outcome: **passed, with two quickstart corrections needed.**

- The command as literally written in quickstart.md has no `--vault`. Run exactly
  as written, `sonavida run --simulate 7 --seed 1 --standins` exits 0 but brings no
  persona alive at all (`$SONAVIDA_HOME/personas/` is never created), because
  `--vault` is what tells `run --simulate` which synthetic definitions to host
  (`contracts/cli.md`). Verified directly: run without `--vault` for 7 simulated
  days, then `ls $SONAVIDA_HOME` — empty. **Proposed correction**: quickstart.md
  line 14 should read `uv run sonavida run --vault "$VAULT" --simulate 7 --seed 1
  --standins`.
- `uv run sonavida memory pellam-quist` (the slug quickstart uses on line 15)
  refuses with `no such persona: pellam-quist`. `cmd_memory` resolves a persona
  either by its memory-file directory name (the persona id, a UUID) or by its
  exact `publicName` (`cli.py:_resolve_persona_memory_path`); it never derives or
  accepts a slug from the definition filename. **Proposed correction**: quickstart
  line 15 should read `uv run sonavida memory "Pellam Quist" | head -40` (or the
  persona id).
- With both corrections, the command ran and printed birth (`self-aware`), the two
  seed memories, and the two shared pasts told in the first person exactly as
  `quickstart.md` expects ("I and someone I once knew crewed opposing boats...").
  With `--seed 1` and the stock `ScriptedModels`/`ScriptedGate` stand-ins (no
  script queued), every turn's model answer is the stand-in's own default
  (`do-nothing`, `PT1H`), so the persona never actually changes presence or goes
  away during the run: quickstart's "every presence change announced... the gap
  when the simulated Studio was off remembered as time away" describes what a
  scripted or real model would produce, not the bare stand-in default. This is not
  a defect — `tests/integration/test_hours.py` proves presence changes and time
  away are handled correctly when the model stand-in is scripted to make them —
  but a first-time reader following quickstart literally will not see either in
  this scenario's output. Worth a one-line note in quickstart.md that scenario 1's
  stand-in default is `do-nothing`, and presence changes/time away are demonstrated
  in the test suite (`test_hours.py`), not by this bare command.

## 2. It makes pieces and names them (US2)

Command run: `uv run sonavida pieces "Pellam Quist"` (same slug-vs-public-name
issue as scenario 1; `pieces pellam-quist` also refuses `no such persona`).

Outcome: **ran, exit 0, but prints nothing** — because, as scenario 1 found, the
plain stand-in default is `do-nothing` every turn, so no piece was ever started in
that run. Quickstart's "Expected: each finished piece has..." presumes a persona
that worked during scenario 1; as written, scenario 1 leaves no piece to list. The
mechanics (`intention`, `attempt`, `title`, `statement` all present on a finished
piece) are independently proven by `tests/integration/test_making.py`, which was
also run as part of T048's `tests/integration/test_week.py`. **Proposed
correction**: either note that scenario 2 needs a scripted model reply sequence (as
in `test_making.py`) to produce a piece, or point scenario 2 at the test instead of
a bare CLI run, the way scenarios 3 to 5 and 8 already do.

## 3. It decides what to show (US3, SC-004)

```bash
uv run pytest tests/integration/test_showing_work.py
```

Outcome: **passed** — 4 passed, 0 failed. Matches quickstart's expectation exactly:
only submitted pieces reach the gate stand-in, only accepted ones reach the Studio
Link stand-in with the gate's labels, rejections return as memories with feedback.

## 4. It remembers what happened, with no numbers (US4, SC-005, SC-006)

```bash
uv run pytest tests/integration/test_experiences.py tests/privacy/test_no_metrics.py tests/privacy/test_erasure.py
```

Outcome: **passed** — 12 passed, 0 failed. Matches quickstart's expectation.

## 5. The Studio's limits are part of its life (US6, SC-007)

```bash
uv run pytest tests/integration/test_studio_limits.py
```

Outcome: **passed** — 2 passed, 0 failed. Matches quickstart's expectation: no
model name, code, or queue position appears in `studio-not-ready`, `interrupted` or
`attempt-failed` memory entries; the `ask` is never lowered, retried or replaced.

## 6. Several personas, each alone (FR-041, SC-010)

Command run, adapted:

```bash
uv run sonavida run --vault "$VAULT3" --simulate 7 --seed 2 --standins \
  --persona pellam-quist --persona ivo-marrowfield --persona sable-quinn
```

`$VAULT3` held `pellam-quist.persona.yaml`, `ivo-marrowfield.persona.yaml` (spec
003 hub examples), and a third synthetic definition written for this run,
`sable-quinn.persona.yaml` (a lightly-edited copy of the `Sable Quinn` fixture
already used by `tests/integration/test_several.py`, id
`00000000-0000-4000-8000-00000000a099`).

Outcome: **ran, exit 0, with one quickstart correction needed.**

- Quickstart line 39 names a third persona `test-persona-nine`, but no file or
  fixture by that name exists anywhere in the hub's spec 003 examples or in
  `components/sonavida/`; searched both trees, no hits. As written, this scenario
  cannot be run — `--persona test-persona-nine` would simply match nothing in a
  two-persona vault, silently hosting only two personas rather than three. It was
  run instead against a synthetic `sable-quinn` definition, adapted from
  `test_several.py`'s own third fixture. **Proposed correction**: quickstart.md
  line 39 should either name and check in a `test-persona-nine.persona.yaml`
  synthetic fixture, or reference `sable-quinn` (or a fixture already present),
  and the prose "seed 2" / "--persona test-persona-nine" combination should be
  checked against whatever fixture is actually chosen.
- With the substitution, three separate memory files were created under
  `$SONAVIDA_HOME/personas/`, one per persona id; `sonavida memory` for each of
  the three public names printed non-empty, entirely separate output (177, 179 and
  171 lines respectively), confirming FR-041/SC-010's "no entry of one in another".
  As in scenario 1, with the stock stand-in default (`do-nothing` every turn), no
  intention was ever formed by any of the three, so "every intention carried out,
  set aside, or waiting" was not exercised by this bare run; that mechanic is
  proven by `tests/integration/test_several.py` and T048's
  `tests/integration/test_week.py`, both of which script real work.

## 7. The team reads, never writes (US5, SC-002)

Only the automated half was run (the human reading trial is T052, explicitly
excluded from this task):

```bash
uv run pytest tests/privacy/test_read_only.py
```

Outcome: **passed** — 4 passed, 0 failed. Confirms `sonavida memory` and
`sonavida pieces` open the memory file `mode=ro`, and that no write path exists.
The ten-question reading trial itself (`tests/fixtures/reading-questions.md`) is
left for T052.

## 8. Leaving (FR-040)

```bash
uv run pytest tests/integration/test_leaving.py
```

Outcome: **passed** — 4 passed, 0 failed. Matches quickstart's expectation: one
`leave-the-museum` choice is remembered as thinking of leaving; a second
consecutive one records the departure, announces away once, and `sonavida run`
refuses to start that persona again.

## Summary

| # | Scenario | Outcome |
|---|---|---|
| 1 | Comes alive, keeps hours | Passed with 2 corrections needed (missing `--vault`; slug vs. public name) |
| 2 | Makes and names pieces | Ran but produced nothing with the bare stand-in default; mechanics covered by `test_making.py` / `test_week.py` |
| 3 | Decides what to show | Passed as written |
| 4 | Remembers, no numbers | Passed as written |
| 5 | Studio's limits | Passed as written |
| 6 | Several personas, each alone | Passed with 1 correction needed (`test-persona-nine` fixture does not exist) |
| 7 | Team reads, never writes (automated half) | Passed as written; human trial deferred to T052 |
| 8 | Leaving | Passed as written |

No proposed correction above was applied to `quickstart.md` itself, per instruction;
they are recorded here for whoever next revises that file.
