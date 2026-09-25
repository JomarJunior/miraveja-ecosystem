# `miraveja-persona`

> Public format tools for persona definitions: load, check, freeze and guard.

A generic library, not a museum component, so it follows the library naming convention (D-033) and carries no brand emoji in its package name.

## Boundaries

- **Owns:** the tools that act on the persona definition format: the bundled schemas, the loaders **🎭 SonaVida** uses, the rule catalog behind `check`, the vault checks and birth ledger, the shared-past listing, the `new` scaffold and the public-repository `guard`.
- **Never owns:** persona content (that lives only in **🔐 CofreAlma**), the format itself (that is the hub's, at `specs/003-cofrealma-persona-definition/contracts/`), or persona behavior.
- **Runs on:** any machine for writing and checking; the Studio for `load_resident`; CI for `guard` and the vault's checks. It never opens a network connection.

## Contracts and dependencies

- **Reads:** a bundled copy of the hub's schemas. CI fails if the copy differs from the hub at the pinned hub commit.
- **Consumed by:** **🎭 SonaVida** (loaders), **🔐 CofreAlma**'s CI (`check --tree`), the lab (`pasts`, the baseline for spec 007), and every public repository's CI (`guard`).

## Constitution rules that bite hardest

- **I:** the `order.*` rules flag schedules, quotas and subject rules; tendencies pass. A definition is read once and frozen, so it cannot steer a living persona.
- **II:** the `metric.*` rules block money and audience numbers anywhere in a definition.
- **VIII:** the guard blocks definitions, author's notes and the vault marker in every public repository and never prints their content. Loader refusals carry a reason code, never prose. Every fixture is synthetic.

## Repository

- Repository: `JomarJunior/miraveja-persona` (public, Apache-2.0). Local: `components/miraveja-persona/`.

## Specs

003 `cofrealma-persona-definition`. Tasks in `specs/003-cofrealma-persona-definition/tasks.md`.
