# The `descridiva` Command Line, version 1

Run on the Studio only. Exit codes: `0` success, `1` the thing asked for did not happen (a refusal or an ended description), `2` usage or configuration error.

| Command | What it does | Exit |
|---|---|---|
| `descridiva serve [--port 8432] [--host 127.0.0.1]` | Runs the service on loopback only (FR-026). Reads `$DESCRIDIVA_HOME/callers.json`, `MODELMORA_URL` (default `http://127.0.0.1:8431`), `MODELMORA_TOKEN` and optional `DESCRIDIVA_MODEL`, written `NAME@VERSION`; any other form is a configuration error. `--host` accepts loopback addresses only and exists so that anything else is refused out loud: the service refuses to start if asked to bind anything but loopback. | 0 · 2 |
| `descridiva callers add NAME --role gate\|viewer` | Registers a caller and prints its token once (R-2). | 0 · 2 |
| `descridiva callers list` | Names and roles; never tokens. | 0 |
| `descridiva callers remove NAME` | Removes a caller. | 0 · 1 |
| `descridiva describe IMAGE [--notes]` | Asks the running service, as the caller whose token is in `DESCRIDIVA_TOKEN`, and prints the description, plus the notes when `--notes` is given and the caller is a gate. Prints the reason on refusal. | 0 · 1 · 2 |
| `descridiva evaluate SET_DIR [--out DIR]` | Team evaluation on the Studio (R-13): describes every image in the set in process against the real **🧠 ModelMora**, compares the notes with the set's manifest, runs the neutrality scan and a determinism check (each image described twice), and writes a review report to `DIR` (default `$DESCRIDIVA_HOME/evaluation-reports/<time>/`). Refuses a `SET_DIR` inside a git working tree (FR-032). | 0 · 1 · 2 |

## The evaluation set manifest

`SET_DIR/manifest.json` lists each image with what the team knows it shows: its subject in a sentence, and for each of the six concerns whether the note should be raised. Images and manifest stay on the Studio. No image in it may show a figure who appears to be or may be a minor with nudity, sexual or suggestive content, or harm (FR-031).
