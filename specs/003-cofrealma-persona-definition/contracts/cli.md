# `miraveja-persona` Interface, version 1

The public surface of the library: a CLI for people and CI, and a Python API for **🎭 SonaVida**. Nothing here opens a network connection (FR-016).

## CLI

| Command | Purpose | Exit codes |
|---|---|---|
| `miraveja-persona new --name "<public name>" [--synthetic] [-o PATH]` | Write a scaffold with a fresh UUID, every part, and a guidance comment for each (R-11). `--synthetic` sets `nature: synthetic`. Refuses to overwrite. | 0 written · 2 usage |
| `miraveja-persona check PATH... [--tree ROOT] [--format text\|json]` | Check definitions and author's notes against the schema and [check-rules.md](./check-rules.md). With `--tree`, also runs the vault-wide and cross-definition rules over the **🔐 CofreAlma** checkout at `ROOT`, reading `decisions.yaml` and `ledger/births.yaml`. | 0 valid (only decided findings) · 1 a certain finding · 3 undecided uncertain findings only · 2 usage or unreadable file |
| `miraveja-persona decide FINGERPRINT --as accepted\|not-an-issue --by NAME [--tree ROOT]` | Record a decision on an uncertain finding in the `decisions.yaml` beside its definition (R-6). Refuses certain findings. | 0 recorded · 1 refused · 2 usage |
| `miraveja-persona pasts ROOT [--format text\|json]` | List every shared past in the vault: story, participants (UUID and public name), telling, and each definition's version (FR-028). Public names are shown because the listing only ever runs against the private vault. The baseline for spec 007. | 0 · 2 usage |
| `miraveja-persona freeze PATH --tree ROOT` | Record a ready definition's birth in `ledger/births.yaml`: identifier, public name, path, SHA-256, date (R-8). Refuses a definition that does not pass `check --tree`, a synthetic one, or one already frozen. | 0 frozen · 1 refused · 2 usage |
| `miraveja-persona guard [PATH...]` | For public repositories: block any definition, author's note or vault marker not marked synthetic, and any secret-shaped value such as a private key, an access token or a password in a URL (R-9, Constitution VIII). A line carrying `guard: fake-secret` holds a deliberate fake and is skipped. Defaults to every file tracked by git in the working directory. Prints file, line and reason only, **never content** and never the secret. | 0 clean · 1 blocked · 2 usage |

`check` prints each finding as:

```text
<file>:<line>  <certainty>  <rule>  <part>
    "<triggering words>"
    breaks <cites>   fingerprint <rule>:<key>:<hash8>
```

`--format json` prints a list of Finding objects (see [data-model.md](../data-model.md)).

## Python API (for **🎭 SonaVida**)

```python
from miraveja_persona import load_resident, load_synthetic, Definition, DefinitionRefused

definition: Definition = load_resident(path, cofrealma_root)   # on the Studio
definition: Definition = load_synthetic(path)                  # tests and examples
```

| Function | Accepts | Refuses with `DefinitionRefused(reason)` |
|---|---|---|
| `load_resident(path, cofrealma_root)` | `nature: resident`, inside a directory marked `.cofrealma`, matching a birth ledger entry, passing every certain rule | `synthetic_as_resident`, `outside_vault`, `not_born`, `changed_since_birth`, `author_note`, `unsupported_version` (names the versions supported), `invalid` (with findings) |
| `load_synthetic(path)` | `nature: synthetic`, passing every certain rule | `resident_outside_studio`, `author_note`, `unsupported_version`, `invalid` |

`Definition` is an immutable model of the schema. It exposes no way to reload, merge or update, since a persona reads its definition once (FR-018). A shared past's `happened` is given with its `{n}` placeholders and its `participants` list; rendering it from the persona's point of view is **🎭 SonaVida**'s job.

Neither loader logs definition content. Refusal reasons and messages never contain prose from the file (Principle VIII).

## Versioning

- The CLI and API follow the format version: `miraveja-persona` 1.x reads format 1.
- A format change that makes an existing definition invalid is format 2 (FR-019), a new schema file in the hub, and a new major version of the library that still reads format 1.
