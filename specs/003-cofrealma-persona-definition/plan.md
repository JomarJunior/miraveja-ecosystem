# Implementation Plan: Resident Persona Definition Format

**Branch**: `003-cofrealma-persona-definition` | **Date**: 2026-09-24 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `/specs/003-cofrealma-persona-definition/spec.md`

## Summary

A persona definition is **one YAML file validated by a closed JSON Schema 2020-12**, kept with that schema in this hub as the public source of truth, with two synthetic example personas and an author's note beside it. Everything that acts on the format sits in one new public Python library, **`miraveja-persona`**. It loads definitions for **🎭 SonaVida**, checks them against a catalog of deterministic rules on the writer's machine and in **🔐 CofreAlma**'s CI, lists every shared past as the baseline for spec 007, freezes a definition into a birth ledger when its persona comes alive, and guards every public repository against leaks. **🔐 CofreAlma** itself stays code-free: it holds definitions, author's notes, check decisions and the birth ledger, and installs the library to check them.

## Technical Context

**Language/Version**: Python 3.12, the same as the Studio's other components.

**Primary Dependencies**: `ruamel.yaml` in safe mode (line numbers for findings, duplicate keys rejected, YAML 1.2 so dates and similar scalars stay strings); `jsonschema` for Draft 2020-12 validation; `pydantic` v2 for the immutable `Definition` model; `argparse` from the standard library for the CLI; `pytest` for tests. Exact versions pinned in `/speckit-tasks`.

**Storage**: plain files. In the hub: the schemas and synthetic examples. In **🔐 CofreAlma**: `personas/<slug>/definition.persona.yaml`, `decisions.yaml` beside each definition, author's notes as `*.note.yaml`, `ledger/births.yaml`, and the `.cofrealma` root marker. The library has no store of its own.

**Testing**: `pytest`. A labeled corpus of synthetic definitions measures the rule catalog (SC-003, SC-007, SC-008); loader tests prove the refusals (FR-022, FR-032); guard tests run in scratch git repositories (SC-004); a network-blocking fixture proves the check sends nothing anywhere (FR-016); a schema-equality test pins the bundled schema to the hub's (R-3).

**Target Platform**: any machine with Python 3.12 for writing and checking; the Studio for `load_resident`; GitHub Actions for guards and **🔐 CofreAlma** CI.

**Project Type**: a library with a CLI, in `components/miraveja-persona/`, plus hub documents.

**Performance Goals**: `check` on one definition under 2 seconds; `check --tree` and `pasts` over 20 definitions under 10 seconds (well within SC-007's minute); `guard` over a component repository under 10 seconds.

**Constraints**: no network use anywhere in the library (FR-016); nothing it prints, logs or raises in `guard` or the loaders contains definition prose (Principle VIII); the check is deterministic, so the same file always gives the same findings; the schema is closed in every object.

**Scale/Scope**: a handful of resident personas at first, a few dozen over time; a few seed memories and shared pasts each.

## Constitution Check

*GATE: passed before Phase 0, re-checked after Phase 1 design.*

| Principle | How this plan complies |
|---|---|
| **I. Personas Are Free Within the Charter** | The schema has no field for a schedule, count, quota, subject rule or lifespan. The `order.*` rules flag prescriptions in prose and cite FR-010, FR-011 or FR-030; tendencies pass. A definition is read once and frozen, so it cannot be used to steer a living persona (FR-018). Every persona-facing constraint in the rule catalog cites its source. |
| **II. Memory, Never Metrics** | `metric.*` rules block money and audience numbers in every prose field, seed memories included. No field can hold a visitor or an experience (`forbidden.visitor`). |
| **III. Two Gates Before Exhibition** | Not weakened: hard-line rules in the check are an early warning for the writer, and the gates still judge everything a persona produces. The check never marks content as allowed for exhibition. |
| **IV. Openly AI, Never Out of Character** | `identity.openlyAI: true` is required; `ai.human-claim` blocks any wording that makes the persona human now or hides that it is an AI (FR-034, FR-035). |
| **V. The Studio Stays Behind the Door** | The library makes no network calls; resident definitions load only from a marked **🔐 CofreAlma** checkout on the Studio (R-10). No model is used by the check (R-4). |
| **VI. One Contract Between Worlds** | Untouched: definitions never cross the Studio Link (spec 001 FR-034). The persona identifier is the same UUID the Studio Link uses, so nothing needs translating. Components still own their own data: **🔐 CofreAlma** holds definitions, **🎭 SonaVida** holds memory. |
| **VII. Quality Over Speed** | Tests come first for the guard and the loader refusals, which are access control. The rule catalog is measured on a labeled corpus before it is trusted. |
| **VIII. Open Code, Private Souls** | The library and format are public; content lives only in **🔐 CofreAlma**. The guard blocks definitions, author's notes and the vault marker in every public repository and never prints their content. Examples are synthetic and marked so. **🔐 CofreAlma** stays code-free. |
| **IX. Frugal by Design** | Plain files, one small library, GitHub Actions already in use. No service, no database, no paid dependency. |

**Post-design re-check (after Phase 1):** passing. Two points were checked closely. First, the check's quotes: `check` prints triggering words, which is right on the writer's machine and in **🔐 CofreAlma**'s private CI, but `guard` runs in public CI, so it prints only file, line and reason (R-9). Second, the loaders' refusal messages: they carry a reason code, never prose, so a refused resident definition cannot leak into **🎭 SonaVida**'s logs.

**Violations:** none.

## Project Structure

### Documentation (this feature)

```text
specs/003-cofrealma-persona-definition/
├── plan.md                  # This file
├── research.md              # Phase 0 output
├── data-model.md            # Phase 1 output
├── quickstart.md            # Phase 1 output
├── contracts/
│   ├── persona-definition-v1.schema.json
│   ├── author-note-v1.schema.json
│   ├── check-rules.md       # the rule catalog
│   ├── cli.md               # CLI and Python API
│   └── examples/            # synthetic personas and an author's note
└── tasks.md                 # /speckit-tasks output, not created here
```

### Source Code

```text
components/miraveja-persona/             new public repo, Apache-2.0
├── src/miraveja_persona/
│   ├── schema/        bundled copies of the hub schemas
│   ├── load.py        safe YAML reading, load_resident, load_synthetic, refusals
│   ├── model.py       the immutable Definition and AuthorNote models
│   ├── rules/         one module per rule family, with its pattern files
│   ├── check.py       runs rules, fingerprints findings, applies decisions
│   ├── vault.py       --tree checks: ledger, reuse, cross-definition shared pasts
│   ├── pasts.py       the shared-past listing
│   ├── scaffold.py    `new`
│   ├── guard.py       public-repository guard
│   └── cli.py
└── tests/
    ├── corpus/        labeled synthetic definitions for SC-003, SC-007, SC-008
    ├── loader/        refusals and single-read behavior
    ├── guard/         scratch git repos; no content in output
    ├── vault/         freezing, reuse, telling mismatches
    └── privacy/       no network, no prose in refusals or guard output

Hub changes (this repository):
├── docs/components/miraveja-persona.md          new component page
├── docs/components/cofrealma.md                 vault layout
├── docs/foundation/ECOSYSTEM-MAP.md             add the library
├── docs/foundation/GLOSSARY.md                  synthetic persona, shared past, author's note, self-image, birth ledger
└── .github/workflows/guard.yml                  the hub runs the guard on itself

Existing public repositories:
└── modelmora, miraveja-studiolink: add `miraveja-persona guard` to CI beside their current guards (FR-021)
```

**Structure Decision**: one new library repository, `JomarJunior/miraveja-persona`, checked out at `components/miraveja-persona/`. Creating it needs the Visionary's go-ahead and is the first task. The split follows the spec's seams: loading for the runtime, the rule catalog for writers, the vault checks for **🔐 CofreAlma**, and the guard for public repositories, so each requirement has one home. **🔐 CofreAlma** gets no code, only the layout above and a CI workflow that installs the library and runs `check --tree`.

## Duties handed to other specs

- **004 `sonavida-persona-life`**: load residents only through `load_resident`, once, at birth; never log definition content; render shared pasts from the persona's point of view; never read author's notes, decisions or the ledger beyond what the loader does.
- **007 `sonavida-persona-society`**: use `miraveja-persona pasts` as the baseline of written relationships and storylines.
- **Every future public component repository**: run `miraveja-persona guard` in CI from its first feature.

## Complexity Tracking

No constitution violations, so nothing to justify.
