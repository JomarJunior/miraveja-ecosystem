# Feature Specification: Model Inference in the Studio

**Feature Branch**: `002-modelmora-inference`

**Created**: 2026-09-23

**Status**: Draft

**Input**: User description: "Give the Studio a single place that owns the open-weight models: which models are available, their name, version and license, and running them for image generation and text generation on one GPU. Other Studio components ask it for a result and get one back, even when several requests arrive at once and the GPU can only hold some models at a time. It must be honest about being busy so callers can wait. Success: a persona runtime and a curator can both request text and images without knowing how models are loaded, and every served model's license is on record."

**Component**: **🧠 ModelMora** (code in `components/modelmora/`)

**Constitution principles touched**: I (Personas Are Free Within the Charter), V (The Studio Stays Behind the Door), VIII (Open Code, Private Souls), IX (Frugal by Design). `/speckit-clarify` and `/speckit-analyze` are recommended, not required.

## Context

The Studio is one machine with one GPU. Every Studio component that needs a generative model goes through **🧠 ModelMora**: **🎭 SonaVida** asks for a persona's thoughts, words and pieces; **🧐 CuraGusta** asks for its judgment; **💬 DescriDiva** asks for descriptions. **🧠 ModelMora** keeps the list of models the Studio may use, with each model's name, version and license, and decides which of them are on the GPU at any moment.

The GPU cannot hold every model at once. Callers must not need to know this. They ask for a result and get one, or they get an honest answer about when they will get one. **🧠 ModelMora** decides nothing about *what* is created. It runs what it is asked to run, as asked.

This spec defines what **🧠 ModelMora** offers its callers and its operators. It does not choose models, formats, or how requests are carried; those are decided in `/speckit-plan`. It is not part of the Studio Link: **🧠 ModelMora** is reachable only inside the Studio.

## User Scenarios & Testing *(mandatory)*

The users of **🧠 ModelMora** are the Studio components that call it (**🎭 SonaVida**, **🧐 CuraGusta**, **💬 DescriDiva**), the team members who decide which models the Studio may run, and the reviewers who must be able to prove every model's license is on record.

### User Story 1 - Ask for text and get it back (Priority: P1)

A team member building **🎭 SonaVida** has a synthetic persona that wants to write a statement for its piece. The runtime sends **🧠 ModelMora** the instructions and the conversation so far, and receives the generated text. It never says which model to load, where, or when. **🧐 CuraGusta** asks for a judgment in exactly the same way.

**Why this priority**: text is how personas think and speak and how the AI gate reasons. Nothing else in Stage A can start without it.

**Independent Test**: with one text model on record and nothing else running, send a text request from a test caller and receive generated text that names the model and version that produced it.

**Acceptance Scenarios**:

1. **Given** a text model is on record and nothing is loaded, **When** a caller asks for text without naming a model, **Then** **🧠 ModelMora** loads the default text model on its own, generates the text, and returns it with the model's name and version.
2. **Given** several text models are on record, **When** a caller names one of them, **Then** the text comes from that model, and the result says so.
3. **Given** a caller names a model that is not on record, **When** it asks for text, **Then** the request is refused with the reason "unknown model", and no other model is used in its place.
4. **Given** a caller asks for text with a limit on its length and a seed, **When** it sends the same request twice to the same model and version, **Then** it receives the same text both times.

---

### User Story 2 - Ask for an image and get it back (Priority: P1)

A synthetic persona has formed an intention for a piece. **🎭 SonaVida** sends **🧠 ModelMora** the description of the image it wants, the size, and optionally a model and a seed, and receives the finished image. The persona's text model may have to leave the GPU to make room. The caller never knows or cares.

**Why this priority**: pieces are images. Without image generation there is nothing to exhibit, and the riskiest assumption (A-006: museum-quality images from one GPU) cannot be tested.

**Independent Test**: with one image model on record, send an image request from a test caller and receive an image, with the model's name, version and the settings actually used.

**Acceptance Scenarios**:

1. **Given** an image model is on record, **When** a caller asks for an image with a description and a size, **Then** it receives an image of that size with the model's name, version, seed and settings used.
2. **Given** a text model occupies the GPU and there is no room for the image model beside it, **When** an image request arrives, **Then** **🧠 ModelMora** makes room and serves the image; the caller sees only a longer wait, never an error about memory.
3. **Given** a caller asks for a size or setting the chosen model cannot produce, **When** it sends the request, **Then** it is refused with a reason that names the problem, before any work is queued.

---

### User Story 3 - Many requests at once, answered honestly (Priority: P1)

Three synthetic personas are awake, and **🧐 CuraGusta** is judging a candidate. Within a few seconds, text and image requests arrive for more models than fit on the GPU together. Each caller learns at once that its request was accepted, where it stands in line, and roughly how long it will wait. Each one later gets its result. Nobody's request is lost, silently dropped, or quietly made smaller to save time.

**Why this priority**: this is the scenario the input names, and it is the everyday state of a Studio shared by several personas and a curator. Principle I requires the Studio's limits to reach personas as honest information they can turn into behavior ("I'll come back to this"), not as hidden throttling.

**Independent Test**: send a burst of mixed text and image requests, for models that cannot all be loaded at once, from several test callers; check that every request receives an immediate answer and later a result, and that the answers match what happened.

**Acceptance Scenarios**:

1. **Given** the GPU is busy, **When** a new request arrives, **Then** the caller is told right away that it was accepted, its position in line, and an estimated wait. The caller does not have to wait with no information.
2. **Given** a caller holds an accepted request, **When** it asks about it, **Then** it learns the request's current state (waiting, running, done, failed, withdrawn), its position if waiting, and an updated estimate.
3. **Given** a request is waiting, **When** its caller withdraws it (for example because the persona went to rest), **Then** it leaves the line, is never run, and the caller is told it was withdrawn.
4. **Given** the line is full, **When** another request arrives, **Then** it is refused as "busy", with an estimate of when to try again. It is never accepted and then dropped.
5. **Given** requests are waiting for several models, **When** **🧠 ModelMora** chooses what to run next, **Then** it follows the ordering rule in FR-019, and no waiting request is overtaken without limit.
6. **Given** any request, **When** its result is returned, **Then** it was produced by the model and settings the request asked for or was told about on acceptance. Quality, size, steps or model are never lowered to relieve load.

---

### User Story 4 - Every served model's license is on record (Priority: P2)

A team member wants the Studio to use a new open-weight model. They add it to **🧠 ModelMora**'s registry with its name, version, license and where it came from, and confirm that the license was read and allows the museum's use. Only then can callers use it. A reviewer can list every model the Studio has ever served, with its license, at any time.

**Why this priority**: the **🧠 ModelMora** component rule requires a record of every served model's name, version and license, and Principle V requires every model to be open-weight. The walking skeleton can run on one pre-recorded model, which is why this is P2 and not P1. Without the registry, though, the feature is not done.

**Independent Test**: add a model with a complete record and see it become available; try to serve a model with an incomplete record and see it refused; list the registry and find every model that produced a result during the test.

**Acceptance Scenarios**:

1. **Given** a team member adds a model with name, version, kind (text or image), license, source and a license confirmation, **When** a caller lists the available models, **Then** the new model appears with its name, version, kind and license.
2. **Given** a model is on record without a license, or without the team member's confirmation, **When** a caller asks for it, **Then** the request is refused, and the model is never loaded.
3. **Given** a model's files on the Studio no longer match what was recorded (a different version), **When** it would be loaded, **Then** it is not served, and the mismatch is reported to the team.
4. **Given** a model is retired from service, **When** a reviewer lists the registry, **Then** the retired model's record, including its license, is still there with the dates it was in service.
5. **Given** any result a caller received in the past, **When** a reviewer looks at the name and version it carries, **Then** the registry holds that exact model's license.

---

### User Story 5 - The Studio's hours are visible to callers (Priority: P3)

The Studio is switched on in the morning. A persona wakes before its models are ready. When the Studio is being shut down, work in progress is told the truth. Callers can always ask whether **🧠 ModelMora** is available and what it can serve, so **🎭 SonaVida** can express the Studio's state as persona behavior ("still waking up", "away from the studio") and not as a failure.

**Why this priority**: Principles I and IV require Studio limits to be expressed as persona states. This needs honest signals from **🧠 ModelMora**, but a first persona can live without the finer points.

**Independent Test**: ask for availability while starting up, while running and while stopping, and check that each answer matches the actual state and that no request is left without an answer on shutdown.

**Acceptance Scenarios**:

1. **Given** **🧠 ModelMora** is starting and no model is ready, **When** a caller asks about availability, **Then** it is told "starting", not "failed".
2. **Given** requests are waiting and **🧠 ModelMora** is told to stop, **When** it stops, **Then** every waiting or running request ends with the answer "stopped before completion". None is left open.
3. **Given** **🧠 ModelMora** is running, **When** a caller asks what it can serve, **Then** it receives the models on record for each kind and the current length of the line, and nothing about the requests of other callers.

---

### Edge Cases

- **A model fails while generating** (for example, it runs out of memory on an unusual request): the request ends with the answer "failed" and a reason the caller can act on. Other requests continue. The same request is not silently retried with lower settings.
- **A model cannot be loaded at all** (files missing, damaged or changed): requests for it are refused with "model unavailable", the team is told, and no other model is used in its place.
- **One request needs more memory than the GPU has, even alone**: refused at once as "cannot be served on this Studio", never queued forever.
- **A model has a built-in content filter that blanks or changes its output**: the result says so. **🧠 ModelMora** does not pass off a filtered output as the requested one, and it adds no content judgment of its own; judging content is the gates' job (Principle III). Explicit work is allowed in the museum when labeled, so a model whose built-in filter cannot be disclosed is not suitable to serve.
- **A caller disappears while its request runs**: the result is kept for a short time for the caller to collect, then discarded. The GPU is not held for it.
- **Two callers ask for exactly the same thing**: they are two requests, and each gets its own result. **🧠 ModelMora** never decides that two requests are the same.
- **A request carries persona definition content** (a persona's prompts and memories, as **🎭 SonaVida**'s requests will): it is used for that request only and never written to logs, errors, the registry or anywhere else that outlives the request (Principle VIII).
- **Someone outside the Studio tries to reach ModelMora**: they cannot. **🧠 ModelMora** answers only callers on the Studio machine (Principle V).
- **A caller asks for a hosted model or a model that is not open-weight**: there is no such thing in the registry, and it cannot be added (Principle V).
- **The registry is empty**: callers are told that no model of that kind is available, not that the service failed.

## Requirements *(mandatory)*

### Functional Requirements

**Serving results**

- **FR-001**: **🧠 ModelMora** MUST generate text from a caller's instructions and prior conversation, and return the text with the name and version of the model that produced it.
- **FR-002**: **🧠 ModelMora** MUST generate an image from a caller's description, size and optional settings (such as a seed or things to avoid), and return the image with the model's name and version and the seed and settings actually used.
- **FR-003**: **🧠 ModelMora** MUST [NEEDS CLARIFICATION: is image understanding (an image plus a question in, text out) in scope for this spec, for **💬 DescriDiva** (roadmap 005) and **🧐 CuraGusta** (roadmap 006)?]
- **FR-004**: A caller MAY name a model on record, or name only the kind of result (text or image) and get the default model for that kind. A persona's choice of model belongs to the persona (Principle I); **🧠 ModelMora** MUST NOT override it.
- **FR-005**: Callers MUST NOT need to load, unload, place or otherwise manage models. **🧠 ModelMora** alone decides what is on the GPU.
- **FR-006**: Given the same model and version, the same request and the same seed, **🧠 ModelMora** MUST return the same result where the model allows it, so pieces and gate decisions can be reproduced.
- **FR-007**: **🧠 ModelMora** MUST NOT substitute a different model, lower a request's size, steps, length or quality, or otherwise change what was asked in order to relieve load. If a request cannot be served as asked, it is refused or fails with a reason. (Principle I)
- **FR-008**: **🧠 ModelMora** MUST NOT decide what is created, and MUST NOT judge, filter or alter content by its own rules. If a model's built-in filter changed an output, the result MUST say so. (Component rule; Principle III places judgment in the gates)

**Sharing one GPU**

- **FR-009**: **🧠 ModelMora** MUST serve requests for more models than fit on the GPU together, loading and unloading models as needed, without any caller seeing a memory error.
- **FR-010**: Every request MUST get an immediate first answer: accepted (with its position in line and an estimated wait), or refused (with a reason).
- **FR-011**: The reasons for refusing or failing MUST be distinct, so a caller can tell "wait and try again" from "this will never work": at least *busy* (with a suggested time to try again), *starting*, *stopping*, *unknown model*, *model unavailable*, *invalid request*, *cannot be served on this Studio*, and *failed during generation*.
- **FR-012**: A caller MUST be able to ask about an accepted request at any time and learn its state (waiting, running, done, failed, withdrawn, stopped before completion), its position if waiting, and an updated estimate.
- **FR-013**: A caller MUST be able to withdraw a request that has not finished. A withdrawn request that has not started MUST NOT run.
- **FR-014**: An accepted request MUST end in exactly one of: a result, a failure with its reason, withdrawn, or stopped before completion. It MUST NOT be dropped or left open forever.
- **FR-015**: The line of waiting requests MUST have a limit. When it is full, a new request MUST be refused as *busy*, and never accepted and then dropped.
- **FR-016**: **🧠 ModelMora** MUST NOT apply quotas, rate limits or priorities to one persona or caller over another beyond FR-019. The Studio's limits reach personas only as the honest answers above, which **🎭 SonaVida** turns into persona behavior. (Principle I)
- **FR-017**: A caller MUST see only its own requests. Nothing it can ask reveals another caller's requests or their content, beyond the length of the line.
- **FR-018**: Estimates MUST be honest: based on how long similar work actually took on this Studio, including any model loading, and updated as the line moves.
- **FR-019**: When choosing the next request to run, **🧠 ModelMora** MUST follow [NEEDS CLARIFICATION: ordering rule. Strictly first come, first served? Group requests for a model already on the GPU, with a limit on how long any request can be overtaken? Or let certain callers (for example the AI gate) go first?]. Whatever the rule, no waiting request MAY be overtaken without limit.

**The registry**

- **FR-020**: **🧠 ModelMora** MUST keep a registry of models. Each record holds: name, version, kind (text or image), license name, where the license terms were read, the source the model came from, a way to confirm the files on the Studio are that exact version, the team member who added it, and when it was added.
- **FR-021**: A model MUST NOT be served unless its record is complete and a team member has confirmed that its license is open-weight and allows its outputs to be exhibited in a public museum. (Principle V; component rule)
- **FR-022**: **🧠 ModelMora** MUST refuse to load a model whose files do not match its recorded version, and MUST tell the team.
- **FR-023**: Retiring a model MUST stop it from being served but MUST keep its record, with the dates it was in service, so every past result can be traced to its license.
- **FR-024**: Callers MUST be able to list the models available to them: name, version, kind, license, and which is the default for each kind.
- **FR-025**: A team member MUST be able to add a model, change the default model for a kind, and retire a model, without changing code.
- **FR-026**: The registry MUST NOT be able to hold a hosted model or anything that runs off the Studio machine. (Principle V)

**Inside the Studio only**

- **FR-027**: **🧠 ModelMora** MUST answer only callers running on the Studio machine, and MUST NOT be reachable from any other machine or network. (Principle V)
- **FR-028**: **🧠 ModelMora** MUST work with no network connection once its models are on the Studio. (Principle V)
- **FR-029**: Callers MUST be able to ask whether **🧠 ModelMora** is starting, running or stopping, and what it can serve. On stopping, every open request MUST be answered *stopped before completion*. (Principles I and IV)

**Private souls**

- **FR-030**: The content of requests and results (instructions, conversation, descriptions, generated text and images) MUST NOT be written to logs, error reports, the registry or any store that outlives the request, except the short holding of a finished result in FR-032. Records of activity MAY hold the caller, the model, the settings, the times and the outcome. (Principle VIII)
- **FR-031**: Tests, examples and fixtures MUST use synthetic personas only. (Principle VIII)
- **FR-032**: A finished result MUST be held for its caller to collect for a limited time (default 1 hour), then discarded.

**Testability**

- **FR-033**: **🧠 ModelMora** MUST offer a test mode with small stand-in models, or none, so callers (**🎭 SonaVida**, **🧐 CuraGusta**, **💬 DescriDiva**) can build and test against it without a GPU and without full-size models.
- **FR-034**: The busy, withdrawal, ordering and shutdown behavior (FR-010 to FR-019, FR-029) MUST be testable without a GPU, using the test mode.

### Key Entities

- **Model record**: one model the Studio may serve, identified by name and version, with its kind, license, source, license confirmation, file check, who added it, when, and its service dates. Kept after retirement.
- **License**: the terms under which a model is used: its name, where its terms were read, and the team member's confirmation that it is open-weight and allows public exhibition of outputs.
- **Default model**: the model on record that serves a kind (text or image) when a caller names no model.
- **Request**: one caller's ask for one result: the kind, the model if named, the inputs, the settings and the caller. Its content lives only as long as the request (FR-030).
- **Request state**: waiting (with position and estimate), running, done, failed, withdrawn or stopped before completion.
- **Result**: generated text or an image, with the model name and version and the settings and seed actually used, and a note if a model's built-in filter changed it.
- **Refusal**: the immediate answer to a request that will not be accepted, with one of the distinct reasons in FR-011.
- **Availability**: whether **🧠 ModelMora** is starting, running or stopping, the models it can serve, and the length of the line.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A test caller acting as **🎭 SonaVida** and another acting as **🧐 CuraGusta** both obtain text and images while naming at most a model and never loading or placing one; zero caller code deals with model loading.
- **SC-002**: In a burst of 20 mixed text and image requests from 4 callers, for more models than fit on the GPU together, 100% of requests end with a result or an explained answer, and none is lost, duplicated or left open.
- **SC-003**: Every request receives its first answer (accepted with position and estimate, or refused with a reason) within 1 second, even while the GPU is fully busy.
- **SC-004**: Once each model has been used at least once, 90% of requests start within 50% of the wait estimated when they were accepted.
- **SC-005**: 100% of the results produced during a test run carry a model name and version whose registry record holds a license, and 100% of attempts to serve a model with an incomplete record are refused.
- **SC-006**: A team member can add a new model to the registry (files already on the Studio) in under 15 minutes, and a reviewer can list every model ever served with its license in under 1 minute.
- **SC-007**: After a test run with synthetic persona requests carrying a unique marker phrase, a search of every log and record **🧠 ModelMora** keeps finds the marker zero times.
- **SC-008**: Across a test run, zero results were produced with a model, size, length or quality setting different from the request or its acceptance answer.
- **SC-009**: Stopping **🧠 ModelMora** with requests open leaves zero requests without a final answer.

## Assumptions

- The callers are **🎭 SonaVida** (roadmap 004), **💬 DescriDiva** (roadmap 005) and **🧐 CuraGusta** (roadmap 006). None exists yet, so this spec is tested with test callers and synthetic personas.
- **🧠 ModelMora** is internal to the Studio. It is not part of the Studio Link and never talks to the Museum side. Model names and versions are Studio internals; callers MUST NOT pass them to visitors (Principle IV). That is the callers' duty, noted here so their specs carry it.
- Explicit and violent work is allowed in the museum when labeled (Principle III), so **🧠 ModelMora** does not filter content. Hard lines are enforced by the two gates, not by the model layer.
- "Honest about busy" means callers always get a clear state and an estimate they can wait on. It does not mean guaranteed times. The estimate is best effort, measured by SC-004.
- Requests live only in memory. If the Studio loses power, open requests are lost; callers see that **🧠 ModelMora** is unavailable and ask again when it returns. Callers keep what they need to resend.
- The size of the line (FR-015), how long a finished result is held (FR-032, default 1 hour) and how long the Studio waits before unloading an idle model are tuned in the plan within Principle IX.
- Which models are chosen, how many fit on the GPU together, how requests are carried between components and how callers on the Studio machine are identified are decided in `/speckit-plan`. The plan must fit one RTX 4090 at no added cost (Principle IX).
- "Open-weight" means the model's weights can be downloaded and run on team hardware. Whether a license's terms allow public exhibition of outputs is judged and confirmed by a team member when adding the model; **🧠 ModelMora** records the judgment and does not make it.
- A model downloaded once is used offline from then on. Fetching models is a team action, not something callers can trigger.
- **Out of scope**: video (a later spec, per the Ecosystem Map), fine-tuning or training, per-persona model adapters, editing an existing image (only generation from a description), streaming partial text, and any use of hosted model APIs (forbidden by Principle V).
