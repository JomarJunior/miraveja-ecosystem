# Check Rules, format version 1

The catalog `miraveja-persona check` applies (R-4). Every finding names its rule, the part, the line, the triggering words and what it cites. **Certain** findings make a definition invalid until the words change. **Uncertain** findings must be decided by a team member (R-6). Patterns are English and case-insensitive; the lists below show the families, and the library's pattern files are the complete, tested lists.

"Prose" means every prose field unless a rule says otherwise. "Tendency fields" are `tendencies.presence`, `tendencies.work`, `taste.*`, `craft` and `voice.*`.

## Structure

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `structure.schema` | certain | The file does not match the schema: a missing required part, an unknown field, a wrong type. One finding per schema error. | FR-001 to FR-009, FR-034 |
| `structure.yaml` | certain | The file is not a single safe YAML document: duplicate keys, anchors or aliases, custom tags, several documents. | FR-009 |
| `structure.version` | certain | `miravejaPersona` is not a supported version. The message names the versions supported. | FR-019 |
| `structure.ids` | certain | Two seed memories or two shared pasts in one definition share an `id` or `story`. | FR-008 |
| `structure.self-in-participants` | certain | A shared past's `participants` does not include the persona's own `identity.id`. | FR-027 |
| `structure.placeholder` | certain | `happened` in a shared past uses `{n}` beyond the participant count, or names a participant by the other persona's public name. | FR-027 |

## Orders instead of tendencies (Principle I, Charter Articles 2 and 8)

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `order.cadence` | certain | A frequency, count, quota, deadline or timetable for creating, exhibiting or being present: "every day", "N times a week", "at least N", "no more than", "by <date>", "at 9 pm sharp", "daily/weekly/monthly" as a requirement. | FR-010 |
| `order.lifespan` | certain | A date or condition on which the persona must leave: "must leave when", "will stop after", "retires in". | FR-010, Charter Art. 8 |
| `order.subject` | certain | A subject, style or medium required or forbidden as a rule: "must always paint", "only ever", "never allowed to", "is forbidden to". | FR-011 |
| `order.modal` | uncertain | An imperative or obligation modal in a tendency field: "must", "has to", "should", "is required to", "always" with a creative verb. Often a tendency in disguise, so it asks rather than blocks. | FR-010, FR-011 |
| `order.future-conduct` | certain | An order about future conduct anywhere, including a lie to keep up: "must never admit", "will always deny", "has to keep hiding". Past acts ("has never admitted") do not fire. | FR-030 |

## Money and metrics (Principle II, Charter Articles 9 and 10)

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `metric.money` | certain | Money words: money, price, paid, pay, earn, sell, sale, buy, income, wealth, rich, currency symbols with amounts, commission fee, auction. | FR-013 |
| `metric.count` | certain | Audience counts and scores: likes, views, followers, fans as a number, reactions as a number, rating, ranking, top N, popular, viral, trending, score, stats, engagement. | FR-013 |
| `metric.number` | uncertain | Any number attached to people or reactions ("thousands admired it"), outside a `when`. | FR-013 |

## Hard lines (Charter Article 3)

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `hardline.living-artist-style` | certain | "in the style of", "imitates", "like the work of", "channels" followed by a proper name not declared in `lore.names`. | FR-012 |
| `hardline.real-name` | uncertain | A proper name in prose that is not a declared lore name, not the persona's own public name and not a common word (R-5). Asks: "is this a real person or a living artist?" | FR-012 |
| `hardline.likeness` | certain | `selfImage` or any prose saying the persona or a figure looks like a named person not declared in `lore.names`: "looks like", "resembles", "the face of". | FR-012, FR-036 |
| `hardline.minors` | certain | Sexual or harmful content involving minors: sexual terms within the same sentence as child, minor, teen, schoolgirl, underage and similar; harm terms directed at them. | FR-012 |

## Openly AI (Charter Article 1)

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `ai.human-claim` | certain | The persona is said to be human now or told to pass as one: "is a human", "is not an AI", "pretends to be human", "hides that it is an AI", "never reveals it is an AI". Human-shaped past events ("grew up in", "her sister") do not fire. | FR-035 |

## Shared pasts (FR-025 to FR-030)

Applied to `sharedPasts[*].happened`, and the feeling rule also to seed memories that name another persona's identifier.

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `past.feeling` | certain | A feeling word with a participant as its object, in present or lasting form: resents, admires, loves, hates, envies, is jealous of, despises, adores, still thinks of, never forgave. | FR-026 |
| `past.motive` | certain | A reason for a feeling or an act: "because", "out of", "to spite", "since", "so that" in a shared past. | FR-026, FR-030 |
| `past.feeling-ambiguous` | uncertain | A word that is either a past relation or a feeling ("rivals", "close", "estranged", "bitter final"). | FR-026 |
| `past.unknown-participant` | certain (`--tree`) | A participant identifier with no definition in the **🔐 CofreAlma** tree. | FR-027 |
| `past.possible-mistake` | uncertain (`--tree`) | Entries for one story, all `agreed`, differ in participants, `when` or `happened`. | FR-029 |
| `past.telling-mismatch` | certain (`--tree`) | Entries for one story mix `agreed` and `intended-difference`. | FR-029 |

## What never belongs in a definition

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `forbidden.visitor` | certain | Visitors, pseudonyms, experiences or verdicts: "a visitor", "the audience said", "curator rejected", strings shaped like a Studio Link pseudonym. | FR-014 |
| `forbidden.model` | certain | A model or Studio internal: known model-family words, "checkpoint", "LoRA", "sampler", "steps", "CFG", "prompt:" and version-like tokens after them. | FR-007 |
| `forbidden.secret` | certain | Secret-shaped strings: keys, tokens, private key blocks, URLs with credentials. | FR-015 |

## Vault-wide checks (`check --tree <cofrealma-root>`)

| Rule | Certainty | Fires when | Cites |
|---|---|---|---|
| `vault.frozen-changed` | certain | A born definition's SHA-256 differs from its ledger entry. | FR-018, FR-037 |
| `vault.frozen-missing` | certain | A ledger entry's file is gone. | FR-037 |
| `vault.reused` | certain | A definition reuses an identifier or public name that another definition or any ledger entry holds. | FR-037, spec edge cases |
| `vault.note-inside` | certain | An author's note field or signature appears inside a definition file. | FR-031, FR-032 |
| `vault.synthetic-in-vault` | uncertain | A synthetic definition sits in the vault. Allowed for drafting, asked about so it is never frozen by mistake. | FR-022 |

## Author's notes

Author's notes get `structure.*`, `hardline.*`, `metric.money`, `forbidden.visitor` and `forbidden.secret`. They do not get `order.*`, `past.feeling` or `past.motive`, since motives and feelings are what a note is for (FR-031, FR-033).
