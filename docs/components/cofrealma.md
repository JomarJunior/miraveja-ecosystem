# **🔐 CofreAlma**

> Private persona definitions for **🖼️ MiraVeja**.

**Etymology:** "Cofre" (Portuguese: safe) + "Alma" (soul). The soul vault.

## Boundaries

- **Owns:** resident persona definitions (identity, taste, voice, tendencies, cares, seed memories, shared pasts), author's notes, check decisions and the birth ledger. The format is spec 003.
- **Never owns:** code.
- **Read by:** **🎭 SonaVida**, on the Studio only.

## Constitution rules that bite hardest

- **VIII:** its contents never enter the hub, a spec, a log, a fixture or any public repository.
- **Component rule:** never checked out under the hub's `components/` or any public location.

## Layout

```text
cofrealma/
├── .cofrealma                              the vault marker; resident definitions load only below it
├── personas/<slug>/
│   ├── definition.persona.yaml             one persona's starting point, frozen once it comes alive
│   └── decisions.yaml                      team decisions on the check's uncertain findings
├── notes/*.note.yaml                       author's notes: team-only lore no runtime ever reads
├── ledger/births.yaml                      one entry per persona that came alive; append only
└── .github/workflows/check.yml
```

- Write a new persona with `miraveja-persona new --name "<public name>" -o personas/<slug>/definition.persona.yaml`.
- Check everything with `miraveja-persona check --tree . personas/*/definition.persona.yaml`. Record a call on an uncertain finding with `miraveja-persona decide`.
- When a persona comes alive, run `miraveja-persona freeze personas/<slug>/definition.persona.yaml --tree .`. From then on its file never changes; anything learned later goes into an author's note.
- List every shared past, the baseline for spec 007, with `miraveja-persona pasts .`.

The vault's CI installs the public library and checks the whole tree:

```yaml
name: check
on: [push, pull_request]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install "git+https://github.com/JomarJunior/miraveja-persona@<pinned commit>"
      - run: miraveja-persona check --tree . personas/*/definition.persona.yaml
```

The check prints the words that triggered each finding. That is fine in this private repository's CI and never acceptable in a public one, which is why public repositories run `miraveja-persona guard` instead: it prints only file, line and reason.

Keep the vault on the Studio. A copy on another machine would also load (spec 003 Assumptions), so where it lives is a team practice.

## Repository

- Repository: `JomarJunior/cofrealma` (**private**, Apache-2.0 license file present but the repository is not published).
- Local: check it out beside the hub, not inside it, for example `~/Developments/Personal/cofrealma`.
- The definition format is specified publicly in the hub with synthetic examples (spec 003). Real definitions are written here, and never anywhere else.

## Specs

003 `cofrealma-persona-definition` (format, in the hub). Prompt in `docs/foundation/SPEC-ROADMAP.md`.
