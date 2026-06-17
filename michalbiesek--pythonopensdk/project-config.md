---
trigger: always_on
description: Full SDK generation rules from OpenAPI
---


# SDK Generation Rules for AI-Driven Generation

> **AI INSTRUCTION:** When asked to generate SDK code from the OpenAPI spec (`openapi.yml`), follow all rules in this file. This is the authoritative source.

Rules and conventions for **AI-driven** generation of the Cribl Python SDK from an OpenAPI specification.

---

## 0. AI Agent Instructions (Read First)

### 0.1 Generation Order

Generate code in this exact order to satisfy dependencies:

1. **types.py** — `Unset`, `UNSET`, `Response`, `File`, `FileTypes`, `RequestFiles`
2. **errors.py** — `UnexpectedStatus`
3. **client.py** — `Client`, `AuthenticatedClient`
4. **models/** — All schemas from `components.schemas` (resolve `$ref` before generating; generate base/referenced schemas first)
5. **api/** — Endpoint modules (each endpoint may depend on models)
6. **api/__init__.py** — Re-exports and client aggregation
7. **__init__.py** — Package exports

### 0.2 Mandatory Rules (NEVER Violate)

- **ALWAYS** use `attrs` for models and clients; **NEVER** use `dataclasses` or raw `__init__`
- **ALWAYS** implement all four endpoint functions: `sync`, `sync_detailed`, `asyncio`, `asyncio_detailed`
- **ALWAYS** use `UNSET` for optional fields; **NEVER** use `None` as "not provided" for JSON-optional fields
- **ALWAYS** include `from __future__ import annotations` at the top of model files
- **ALWAYS** use relative imports (`from ...client`, `from ...models.x import Y`)
- **NEVER** generate placeholder or TODO code; generate complete, runnable code
- **NEVER** invent schema properties; only use properties defined in the OpenAPI spec
- **NEVER** add `__pycache__/` or `*.pyc` binary files to the repository; ensure `.gitignore` excludes them

### 0.3 Schema Name -> File Name Mapping

- PascalCase schema name -> snake_case filename: `HealthServerStatus` -> `health_server_status.py`
- Class name = schema name (PascalCase)

### 0.4 When Parsing OpenAPI

- For each `operationId`, derive the endpoint module name from the path and method (e.g. `get_health` for `GET /health`)
- For each response status code with a schema, add a branch in `_parse_response`
- For `oneOf`/`anyOf` with discriminator: check discriminator field first, then instantiate the correct subclass

### 0.5 Prompt Context for AI

When invoking an AI agent to generate SDK code, provide:

```text
Generate Python SDK code following this Cursor rule.
Source: openapi.yml (OpenAPI 3.1)
Target: [package_name] (e.g. cribl_api_reference_client)
Scope: [all | models only | api only | specific path, e.g. /health | by tag, e.g. auth,health]
```

For incremental generation, specify scope (e.g. `by tag: auth,health`) to avoid regenerating the entire SDK. For large specs (50k+ lines), generate by tag or path prefix in batches.

### 0.6 Post-Generation Validation (AI Self-Check)

Before finishing, verify:

1. All `$ref` in schemas are resolved; no forward references to undefined models
2. Every endpoint has exactly four functions: `sync`, `sync_detailed`, `asyncio`, `asyncio_detailed`
3. `_parse_response` has an `if response.status_code == X` branch for every documented response
4. Optional request/response fields use `UNSET`, not `None`, for "not provided"
5. Python reserved words (`id`, `object`, etc.) are escaped (e.g. `id_` or `object_`) in parameter names
6. No `# TODO` or placeholder implementations
7. No `__pycache__/` or `*.pyc` files in the repository; `.gitignore` excludes them

### 0.7 Common AI Pitfalls to Avoid

| Pitfall | Correct Behavior |
|---------|------------------|
| Using `None` for optional JSON fields | Use `UNSET`; `None` means "explicitly null" in JSON |
| Omitting `asyncio`/`asyncio_detailed` | Always generate all four endpoint functions |
| Hardcoding base URL in endpoints | Use `client.base_url` or accept `server_url` override |
| Inventing response schemas | Only use schemas from `components.schemas` and `responses.*.content.*.schema` |
| Skipping `additional_properties` | If schema has `additionalProperties`, add the dict + accessor methods |
| Wrong import depth | Use `...client` from `api/health/`, `..models` from `api/` |
| Discriminator not checked first | In `from_dict` for unions, check discriminator before instantiating |

---

## 1. Reference & Overview

- **OpenAPI**: 3.1.0, Cribl API Reference
- **Python**: >=3.10, <4
- **Generator-agnostic**: These rules do not depend on gen.yaml or any specific code generator config.

---

## 2. OpenAPI Specification Requirements

### 2.1 Required Vendor Extensions

| Extension | Purpose |
|-----------|---------|
| `x-retries` (or equivalent) | Backoff retry metadata for 429 |
| `x-group` (or equivalent) | Resource grouping (e.g. `auth.tokens`, `nodes`) |
| `x-name-override` (or equivalent) | Friendly method name (e.g. `get`, `list`, `create`) |

### 2.2 Security Schemes

- **bearerAuth**: HTTP Bearer (JWT) for on-prem
- **clientOauth**: OAuth2 client credentials for Cribl.Cloud
- Vendor-specific token endpoint metadata for OAuth audience

### 2.3 Base URL Contexts

Document these in `info.description`:

- Leader: `/api/v1`
- Worker Group / Edge Fleet: `/api/v1/m/{groupName}`
- Host (Worker/Edge Node): `/api/v1/w/{nodeId}`
- Search: `/api/v1/m/default_search`

---

## 3. Project Structure

```text
project/
├── openapi.yml                 # Source OpenAPI spec

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michalbiesek/pythonOpenSDK](https://github.com/michalbiesek/pythonOpenSDK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
