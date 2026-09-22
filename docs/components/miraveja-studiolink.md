# `miraveja-studiolink`

> The runnable side of the Studio Link: a client, a reference stand-in and a conformance suite.

A generic library, not a museum component, so it follows the library naming convention (D-033) and carries no brand emoji in its package name.

## Boundaries

- **Owns:** message models for the Studio Link, the Studio-side client, the reference stand-in (a conforming in-memory Museum side), and the conformance suite that checks any Museum end.
- **Never owns:** the contract itself (that is the hub's, at `specs/001-studiolink-contract/contracts/studiolink-v1.yaml`), persona behavior, curation, museum storage.
- **Runs on:** anywhere Python runs. The stand-in is for development and tests, never for visitors.

## Contracts and dependencies

- **Reads:** the hub's contract document, from the hub checkout it sits inside, overridable with `STUDIOLINK_CONTRACT_PATH`. CI pins a hub commit rather than copying the file.
- **Consumed by:** **🎭 SonaVida** and **🧐 CuraGusta** (the Studio end), **🏛️ MuseuMusa** (checked by the conformance suite).

## Constitution rules that bite hardest

- **II:** the client refuses any message from the Museum side carrying an unknown field, so no metric can arrive unnoticed.
- **V:** the client only ever initiates; nothing here listens for the Museum side.
- **VII:** contract tests are written before either end.
- **VIII:** every fixture and example uses synthetic personas; CI fails on a real persona definition, name or secret.

## Repository

- Repository: `JomarJunior/miraveja-studiolink` (public, Apache-2.0). Local: `components/miraveja-studiolink/`.
- **Not created yet.** Creating it is task T001 of spec 001 and needs the Visionary's go-ahead.

## Specs

001 `studiolink-contract`. Tasks in `specs/001-studiolink-contract/tasks.md`.
