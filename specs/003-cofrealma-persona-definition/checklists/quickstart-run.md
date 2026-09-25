# Quickstart Run: spec 003

**Run**: 2026-09-25, against a scratch vault built from the hub's synthetic examples, with `miraveja-persona` at commit `78aea21`.

| # | Scenario | Result |
|---|---|---|
| 1 | Hub examples are valid | Pass: exit 0, no findings |
| 2 | Scaffold for a new persona | Pass: `new` writes it; `check` exits 1 naming each empty required part. The timed trial itself (SC-001, T018) needs a team member and is still open |
| 3 | Orders, metrics, hard lines, human claims | Pass: "Posts three pieces every day" gives `order.cadence`, exit 1; the labeled corpus (87 cases, 10 of them complete clean definitions) passes with full recall and no clean definition carrying a false finding |
| 4 | Shared pasts | Pass: tree check exits 3 with only `vault.synthetic-in-vault`; `pasts` lists `junior-regatta` agreed and `lighthouse-mural` as an intended difference; feeling, differing text, mixed markings, an order to keep a lie and an unknown participant each give their rule |
| 5 | Uncertain findings are decided | Pass: "bitter" gives `past.feeling-ambiguous`, exit 3; after `decide`, exit 0 with the finding shown as decided |
| 6 | Birth, freezing and reuse | Pass: `freeze` records the birth; one changed word gives `vault.frozen-changed`; a copy reusing the identifier and name gives `vault.reused` |
| 7 | Private by design | Pass: guard tests pass; the hub itself passes `guard`; all seven other public repositories run it in CI |
| 8 | The runtime reads the definition alone | Pass: loader tests pass, including every refusal code |

Two quickstart lines were corrected to match the built behavior: scenario 4 now expects exit 3 from the synthetic-in-vault question, and scenario 5 uses "bitter" rather than "rivals", which passes as a past fact (FR-025).
