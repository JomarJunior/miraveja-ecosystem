# Quickstart: Resident Persona Definition Format

Runnable scenarios that prove spec 003 works. Commands assume `miraveja-persona` is installed from `components/miraveja-persona/` (`pip install -e components/miraveja-persona`) and are run from the hub root unless stated. Interface details are in [contracts/cli.md](./contracts/cli.md); rules in [contracts/check-rules.md](./contracts/check-rules.md).

Every scenario uses the synthetic personas in [contracts/examples/](./contracts/examples/) or scratch copies of them. **Never run these steps against a real definition outside the private vault.**

## Prerequisites

- Python 3.12.
- For vault scenarios: a scratch directory set up like the **🔐 CofreAlma** checkout (below). The real vault is never used for these checks.

```bash
VAULT=$(mktemp -d)
touch "$VAULT/.cofrealma"
mkdir -p "$VAULT/personas/pellam" "$VAULT/personas/ivo" "$VAULT/notes" "$VAULT/ledger"
cp specs/003-cofrealma-persona-definition/contracts/examples/pellam-quist.persona.yaml "$VAULT/personas/pellam/definition.persona.yaml"
cp specs/003-cofrealma-persona-definition/contracts/examples/ivo-marrowfield.persona.yaml "$VAULT/personas/ivo/definition.persona.yaml"
cp specs/003-cofrealma-persona-definition/contracts/examples/lighthouse-mural.note.yaml "$VAULT/notes/"
```

## 1. The synthetic example is valid (FR-023, User Story 1)

```bash
miraveja-persona check specs/003-cofrealma-persona-definition/contracts/examples/*.yaml
```

Expected: exit 0, no findings.

## 2. A new persona in under an hour (SC-001)

```bash
miraveja-persona new --name "Test Persona Nine" --synthetic -o /tmp/nine.persona.yaml
miraveja-persona check /tmp/nine.persona.yaml
```

Expected: the scaffold fails with `structure.schema` findings naming each empty required part. A team member who has not seen the format fills it in with the example open, and times it. `check` reaches exit 0 within the hour.

## 3. Orders, metrics, hard lines and human claims are flagged (SC-003, User Story 3)

```bash
pytest components/miraveja-persona/tests/corpus -q
```

The labeled corpus holds synthetic definitions seeded with each rule family and a set of clean ones. Expected: 100% of seeded violations found; at most one false finding per clean definition; each finding names part, line, words and cites.

Spot check by hand: add `tendencies.work: "Posts three pieces every day."` to a copy of the example and run `check`. Expected: exit 1, `order.cadence`, citing FR-010.

## 4. Shared pasts: facts pass, feelings and orders do not (SC-007, SC-008)

```bash
miraveja-persona check --tree "$VAULT" "$VAULT"/personas/*/definition.persona.yaml
miraveja-persona pasts "$VAULT"
```

Expected: exit 0. `pasts` lists `junior-regatta` as agreed with one version, and `lighthouse-mural` as an intended difference with two versions.

Then, in the scratch vault:

| Change | Expected |
|---|---|
| Append "{1} still resents {2} for it." to Pellam's `junior-regatta` | exit 1, `past.feeling` |
| Change Ivo's `junior-regatta` text only | exit 3, `past.possible-mistake` (uncertain) |
| Mark Pellam's `lighthouse-mural` as `agreed` | exit 1, `past.telling-mismatch` |
| Add a seed memory to Ivo: "I must never admit who painted the gull." | exit 1, `order.future-conduct` |
| Add a participant UUID with no definition | exit 1, `past.unknown-participant` |

## 5. Uncertain findings are decided, never passed silently (FR-017)

Add the word "rivals" to a copy's shared past. `check` exits 3 with `past.feeling-ambiguous`. Run `miraveja-persona decide <fingerprint> --as accepted --by tester --tree "$VAULT"`. Expected: `check` exits 0 and still reports the finding as decided.

## 6. Birth, freezing and reuse (FR-018, FR-037)

Use a copy of the example with `nature: resident` in the scratch vault only.

```bash
miraveja-persona freeze "$VAULT/personas/pellam/definition.persona.yaml" --tree "$VAULT"
```

Expected: a ledger entry. Then edit one word of the frozen file: `check --tree` exits 1 with `vault.frozen-changed`, and `load_resident` refuses with `changed_since_birth`. A new definition reusing Pellam's UUID or public name exits 1 with `vault.reused`.

## 7. Private by design (SC-004, User Story 4)

```bash
pytest components/miraveja-persona/tests/guard -q
```

Expected: in a scratch git repository, the guard blocks a file marked `nature: resident`, an unmarked definition, an author's note marked resident, a pasted `miravejaPersona:` line inside a Markdown file, and a `.cofrealma` marker, and passes the hub's synthetic examples. Its output never contains a line of the blocked file's content.

The hub and every public component repository run `miraveja-persona guard` in CI; a pull request adding a resident definition fails there.

## 8. The runtime reads the definition alone (SC-002, User Story 2)

```bash
pytest components/miraveja-persona/tests/loader -q
```

Expected: `load_synthetic` returns every part of each example with no other input, and swapping one example for the other changes only the data. `load_resident` refuses a synthetic definition (`synthetic_as_resident`), a definition outside a marked vault (`outside_vault`), one not in the ledger (`not_born`), and any author's note (`author_note`). `load_synthetic` refuses a resident one (`resident_outside_studio`).
