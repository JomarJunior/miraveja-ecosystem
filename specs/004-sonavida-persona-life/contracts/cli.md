# `sonavida` Command Line, version 1

Runs on the Studio only. Every command reads **🔐 CofreAlma** read-only and writes only under `$SONAVIDA_HOME`.

| Command | Purpose | Exit codes |
|---|---|---|
| `sonavida run --vault ROOT [--persona SLUG ...]` | Bring to life every frozen resident persona in the vault that is not alive and has not left (or only those named), and keep them living until stopped. On SIGINT/SIGTERM, announce each as away and stop in order (FR-008). | 0 stopped in order · 1 refused (see stderr) · 2 usage |
| `sonavida run --simulate DAYS --seed N --standins [--vault ROOT]` | Test mode: simulated clock, stand-ins for **🧠 ModelMora**, perception, the gate and the Studio Link reference stand-in (FR-038, FR-039). Synthetic personas only; `--vault` points at a scratch vault of synthetic definitions. | 0 · 1 · 2 |
| `sonavida run --vault ROOT --dry-run [--persona SLUG ...]` | Pre-alpha run on the Studio: real **🧠 ModelMora** on loopback, real clock, real memory, with the perception and gate stand-ins and the in-process Studio Link reference stand-in, so nothing can reach a Museum side (Principle III, R-9). Resident personas must be frozen, as for a real run. | 0 · 1 · 2 |
| `sonavida memory PERSONA [--from DATE] [--to DATE]` | Print the persona's memory in time order, plain language, each decision beside its reason (FR-035). Read-only. | 0 · 2 |
| `sonavida pieces PERSONA` | List the persona's pieces, their state history, titles, statements and labels. Read-only. | 0 · 2 |
| `sonavida status` | List personas: alive, resting, away, or departed. No counts of anything the personas did. | 0 |

Refusals from `run`, each with a reason code and never any definition prose: `synthetic_as_resident`, `outside_vault`, `not_born`, `changed_since_birth`, `author_note`, `already_alive` (FR-003, FR-004), `departed` (FR-040), `gate_not_configured` (a real Museum side needs a real AI gate; the gate stand-in works only with `--simulate --standins`, Principle III).

There is no command, flag or API that writes to, edits or deletes from a persona's memory (FR-036), and no network listener of any kind.
