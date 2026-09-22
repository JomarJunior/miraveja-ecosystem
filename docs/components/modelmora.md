# **🧠 ModelMora**

> Open-weight model management and inference for the **🖼️ MiraVeja** Studio.

**Etymology:** "Model" + "Mora" (from Portuguese "morar": to live). Where the models live.

## Boundaries

- **Owns:** the model registry (name, version, license), loading and unloading, GPU scheduling, inference for image and text (video later).
- **Never owns:** any decision about what to create.
- **Runs on:** the Studio (RTX 4090).

## Contracts and dependencies

- **Exposes:** an inference interface inside the Studio.
- **Consumed by:** **🎭 SonaVida**, **💬 DescriDiva**, **🧐 CuraGusta**.
- **Depends on:** nothing else in the ecosystem.

## Constitution rules that bite hardest

- **V:** open-weight models on team hardware only; no hosted model APIs.
- **IX:** one GPU; share it honestly (tell callers when busy).
- **Component rule:** record every served model's name, version and license.

## Repository

- Repository: `JomarJunior/modelmora` (public, Apache-2.0). Local: `components/modelmora/`.
- The earlier ModelMora is archived privately as `JomarJunior/modelmora-legacy`. Its code is not carried over.
- To do in the first feature: README headed **🧠 ModelMora** linking to the hub; persona-definition and secrets check.

## Specs

002 `modelmora-inference`. Prompt in `docs/foundation/SPEC-ROADMAP.md`.
