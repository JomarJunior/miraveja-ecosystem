# Ports, version 1

The seams where real components replace stand-ins without changing a persona's life (FR-038).

| Port | Real implementation | Stand-in | Contract it follows |
|---|---|---|---|
| `Models` | **🧠 ModelMora** over loopback HTTP; the client refuses any non-loopback host (Principle V) | scripted: answers text, images, *starting*, *busy*, *stopping*, *failed* | `specs/002-modelmora-inference/contracts/modelmora-v1.yaml` |
| `Perception` | **💬 DescriDiva** (roadmap 005) | asks `Models` for text about the image with a neutral instruction; or scripted descriptions in tests | returns one neutral description per image |
| `AiGate` | **🧐 CuraGusta** (roadmap 006) | returns accepted/rejected, reason, labels (persona's suggestion plus scripted), feedback on rejection; on acceptance hands the candidate to `StudioLink` | returns a Verdict as spec 001 defines it |
| `StudioLink` | `miraveja-studiolink` client against the Museum side | the same client against the reference stand-in | `specs/001-studiolink-contract/contracts/studiolink-v1.yaml` |
| `Vault` | `miraveja_persona.load_resident` and `pasts.list_pasts` over **🔐 CofreAlma** | the same over a scratch vault of synthetic personas (tests load with `load_synthetic`) | `specs/003-cofrealma-persona-definition/contracts/cli.md` |
| `Clock` | Studio local time | simulated, seeded | — |

Every port answer that reaches a persona is translated into its life (turn-protocol.md, R-7); none of the ports' own words, codes or model names reach memory.
