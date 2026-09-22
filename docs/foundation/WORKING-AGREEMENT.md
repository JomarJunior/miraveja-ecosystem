# **🖼️ MiraVeja** Working Agreement

How the team and Claude Code build **🖼️ MiraVeja** with Spec Kit. Verified against Specify CLI 1.0.9 on 2026-09-21.

## The setup

- Run Claude Code from the hub root (`miraveja-ecosystem/`). Spec Kit commands are skills: `/speckit-<command>`.
- Component repositories are checked out under `components/<name>/`. The hub ignores them. **🔐 CofreAlma** is never checked out here.
- Specs are always created in the hub's `specs/`, even if a command is run from inside a component (verified).
- Spec Kit's git extension is not installed. Branches and commits in component repositories are made by hand or by Claude Code on request.

## The loop, per feature

| Step | Command | Required? |
|---|---|---|
| 1 | `/speckit-specify <prompt from SPEC-ROADMAP.md>` | Always |
| 2 | `/speckit-clarify` | Required if the spec touches Principles II, III, IV or VI; otherwise recommended |
| 3 | `/speckit-plan <tech choices for this component>` | Always. The plan must include a Constitution Check and state which `components/<name>/` it writes to |
| 4 | `/speckit-checklist` | Optional |
| 5 | `/speckit-tasks` | Always. Tests come before implementation for gates, the contract and access control |
| 6 | `/speckit-analyze` | Required if the spec touches Principles II, III, IV or VI; otherwise recommended |
| 7 | `/speckit-implement` | Always. Code goes into `components/<name>/` |
| 8 | `/speckit-converge` | Always. Repeat implement and converge until it reports converged |

Then: commit in the component repository with the spec number in the message, and commit the spec folder in the hub.

## Where technology is decided

Specs say what and why. Technology is chosen in `/speckit-plan`, per component, within the constitution (open-weight models only, one GPU machine, one small server, no added cost).

## When to open a new spec

- A new capability, or a change a visitor or persona would notice.
- Any change to the Studio Link contract (always its own spec, with a version bump).
- A bug fix that changes agreed behavior. Plain bugs are fixed without a spec.

## When to amend the constitution

- A principle blocks a good decision, or a new lasting rule emerges from repeated plan discussions.
- A new paid dependency is proposed (Principle IX).
- Use `/speckit-constitution`, get the Visionary's approval, bump the version, and record the amendment in `FOUNDATION-LOG.md`.

## When to amend the Museum Charter

- A persona-facing rule changes, or an open item is decided.
- Edit `docs/MUSEUM-CHARTER.md`, bump its version, review it against Principles I and III in the same change, and record it in `FOUNDATION-LOG.md`.

## Following the build order

Work follows `ECOSYSTEM-MAP.md`: Stage A (Studio alone, private) before Stage B (doors open). A feature outside the current stage needs the map updated first.

## Every public component repository

- Apache License 2.0 (already in place).
- A README headed with the component's emoji and bold name, linking back to the hub.
- An automated check that blocks persona definitions and secrets (Principle VIII), added as part of the repository's first feature.
