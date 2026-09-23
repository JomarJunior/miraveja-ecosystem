# Phase 1 Data Model: Model Inference in the Studio

Two kinds of state: **durable** (the SQLite registry, which outlives every request) and **live** (the line and its results, which exist only while **🧠 ModelMora** runs). FR-030 draws the line between them: no request or result content may ever enter the durable side.

## Durable: the registry (SQLite)

### `model`

| Field | Type | Rules |
|---|---|---|
| `id` | integer | Surrogate key. |
| `name` | text | Together with `version`, unique. |
| `version` | text | The exact version served. |
| `kind` | text | `text` or `image`. |
| `reads_images` | boolean | Text models only; false for image models (FR-003). |
| `license_name` | text | Required. |
| `license_source` | text | Where the terms were read. |
| `source` | text | Where the model came from. |
| `weights_digest` | text | Verifies the files are that version (FR-022, R-8). |
| `added_by` | text | The team member who added it. |
| `added_at` | timestamp | |
| `license_confirmed_by` | text | The team member who read the license (FR-021). |
| `license_confirmed_at` | timestamp | |

A record is **complete** only with a license name, a license source and a confirmation. Anything less is never served (FR-021).

### `service_period`

| Field | Type | Rules |
|---|---|---|
| `model_id` | integer | |
| `from` | timestamp | When it entered service. |
| `to` | timestamp, nullable | `null` while in service; set on retirement. Records are kept forever (FR-023). |

### `default_model`

| Field | Type | Rules |
|---|---|---|
| `slot` | text | `text`, `text_with_images` or `image` (FR-003, FR-024). |
| `model_id` | integer | Must point at a complete, in-service record. |

**Cannot be represented:** a hosted model. There is no field for an endpoint, and every record requires local weights with a digest (FR-026).

## Live: requests and results

### Request

| Field | Type | Rules |
|---|---|---|
| `request_id` | uuid | Returned on acceptance. |
| `caller` | text | Which Studio component asked (R-9). A caller sees only its own (FR-017). |
| `kind` | `text` \| `image` | |
| `model` | name and version, optional | Absent means the default for the kind (FR-004). Never substituted (FR-007). |
| `inputs` | instructions, prior turns, optional images (text) or a description (image) | Content lives only as long as the request (FR-030). |
| `settings` | seed, length limit, size, things to avoid, steps | Refused before queueing when the model cannot honor them. |
| `submitted_at` | timestamp | Sets arrival order (FR-019). |
| `state` | see below | |

### Request state

```
                 ┌── refused (never accepted: busy, unknown model, invalid request, …)
submit ──────────┤
                 └── accepted → waiting ──→ running ──→ done
                                  │            │
                                  │            ├──→ failed
                                  ├──→ withdrawn (FR-013)
                                  └──→ stopped before completion (FR-029)
```

Every accepted request ends in exactly one of `done`, `failed`, `withdrawn` or `stopped_before_completion` (FR-014). A `waiting` request also carries its position and the current estimate (FR-012, FR-018).

### Result

| Field | Type | Rules |
|---|---|---|
| `request_id` | uuid | |
| `model_name`, `model_version` | text | Always present, so every output traces to a license (SC-005). |
| `text` or `image` | text, or bytes plus a media type | |
| `settings_used` | seed, size, steps, length | What actually ran, not what was hoped for (FR-002, FR-008). |
| `filter_note` | text, nullable | Set when a model's own built-in filter changed the output (FR-008). |
| `held_until` | timestamp | Discarded after the holding time (FR-032, default 1 hour). |

### Refusal reasons (FR-011)

`busy` (with a suggested retry time) · `starting` · `stopping` · `unknown_model` · `model_unavailable` · `invalid_request` · `cannot_be_served_on_this_studio` · `failed_during_generation`

The first three say "wait"; the rest say "this will not work as asked".

### Availability

`state` (`starting`, `running`, `stopping`), the models servable per kind with their defaults, and the current length of the line — never anything about another caller's requests (FR-017, FR-029).

## Residency (in the worker, not on the wire)

| Concept | Meaning |
|---|---|
| **Resident model** | Loaded on the GPU, with its measured memory footprint and last-used time. |
| **Eviction** | Unloading the least recently used resident model to make room. Invisible to callers, who see only a longer wait (FR-009). |
| **Idle unload** | A resident model unused beyond the idle timeout is unloaded to leave the GPU free. |
| **Overtaking budget** | How long a waiting request has been passed over by newer requests for resident models. Once it reaches the bound, that request runs next (FR-019). |

## What is deliberately absent

- Any per-caller quota, priority or rate limit (FR-016).
- Any record of request or result content in the registry, logs or errors (FR-030).
- Any content judgment or safety verdict of **🧠 ModelMora**'s own (FR-008).
- Any field that could point at a model running off this machine (FR-026).
