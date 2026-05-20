---
trigger: always_on
description: > **Scope:** This file tailors GitHub Copilot (Chat/Code/Review) to this monorepo. It encodes *non-negotiable* conventions so Copilot generates code that builds, tests, and deploys with our Tilt/Helm setup.
---

# .github/copilot-instructions.md

> **Scope:** This file tailors GitHub Copilot (Chat/Code/Review) to this monorepo. It encodes *non-negotiable* conventions so Copilot generates code that builds, tests, and deploys with our Tilt/Helm setup.

---

## 1) Repository at a glance

* **Monorepo:** Python services + Vue 3/TypeScript frontend
* **Services:** `services/rag-backend`, `services/admin-backend`, `services/document-extractor`, `services/mcp-server`, `services/frontend`
* **Libs:** Reusable APIs/logic in `libs/*-api` and `libs/*-lib` consumed by services
* **Infra:** `infrastructure/rag` (Helm chart, K3d/Tilt config, Terraform). Local orchestration via **Tilt**; env loaded from `.env`.
* **Vector DB & extras:** Qdrant, KeyDB/Redis, Langfuse (observability). LLM provider via OpenAI-compatible APIs (STACKIT/ollama).

**Rule of thumb:** *Business logic lives in **libs**; services are thin assemblies (routing/DI/bootstrap only).*

---

## 2) Languages & toolchain

* **Python:** 3.13, **FastAPI**, **Pydantic v2**, **Poetry**
* **Formatting/Linting:** `black` (line length **120**), `isort` (profile `black`), `flake8` (plugins configured in repo)
* **Testing:** `pytest` (+ coverage), prefer `pytest-asyncio` for async
* **Frontend:** Vue 3, TypeScript, Vite, Nx, Tailwind, Pinia, Vue I18n; tests via Vitest + Cypress; lint via ESLint

---

## 3) Copilot behavior (important)

* **Follow this file strictly.** Do not invent secrets/URLs/paths. Use placeholders/TODOs.
* **Prefer libs over services:** New endpoints and business logic go to `libs/*-api` & `libs/*-lib`; services only import/wire.
* **Always emit tests** with new Python code (1 happy + 1 edge case).
* **Ask via comments if uncertain**: Add a short `# Assumptions:` block at top of the diff.

---

## 4) Python conventions

* **Typing:** Use modern typing (`list[str]`, `dict[str, Any]`), avoid `Any` unless necessary. Pydantic v2 validators (`field_validator`, `model_validator`).
* **Imports:** **Absolute imports only** (no `from .x import y`). Group per isort. No wildcard imports.
* **FastAPI:**

  * Endpoints are **thin**; DI via `Depends`/container; no business logic in routers.
  * Every route sets `response_model=...`; sanitize outputs.
  * Centralize exception mapping; do not leak stack traces in prod.
  * Prefer **async** endpoints for I/O-bound paths.
* **HTTP client:** Use `httpx.AsyncClient` with timeouts/retries. **Never** use `requests` in async code.
* **Config:** Use environment variables via `pydantic-settings` (document required vars; load `.env` in dev tooling only). Do **not** hardcode secrets.
* **Logging:** Use `logging` (or `structlog` if present). Produce structured, contextual logs (`event`, `service`, `request_id`). Do not log PII.
* **Repositories/Adapters:** Implement external I/O (DBs, HTTP, object storage) behind repository interfaces. Services call repositories; routers call services.
* **File layout for new code:**

  * `libs/<package>/src/<package>/...` → APIs, services, repositories, schemas, utils
  * `libs/<package>/tests/` → tests for the above
  * `services/<service>/` → service bootstrap, DI container, runtime config

**Flake8 style (align with repo):** max line length **120**; `max-complexity: 8`; `annotations-complexity: 3`; `ignore: E203, W503, E704`; **double quotes** for code & docstrings; docstrings **NumPy style**; absolute imports only.

---

## 5) Testing

* **pytest** with fixtures/mocks (`unittest.mock`, `pytest-mock`).
* Keep unit tests fast and isolated; integration tests go through defined interfaces.
* No real network in unit tests; for `httpx` use `respx` (or equivalent) to mock.
* Target **high coverage on core libs**; measure with coverage in CI.

**pytest.ini (suggested):**

```ini
[pytest]
addopts = -q --strict-markers --strict-config --maxfail=1 --cov=libs --cov-report=term-missing
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
asyncio_mode = strict
```

---

## 6) Frontend quick rules

* Use `<script setup lang="ts">`, Composition API; strong typing (avoid `any`).
* State via **Pinia**; i18n via **Vue I18n**. Small, presentational components; move logic to composables/stores.
* Use Tailwind utilities; avoid inline styles. Tests via **Vitest** (unit) and **Cypress** (E2E).
* Use tsconfig path aliases (e.g., `@shared/utils`); avoid deep relative chains.

---

## 7) Infra & local dev hooks (read-only for Copilot)

* Local dev via **Tilt** (`tilt up`); initial `helm dependency update` under `infrastructure/rag` may be required.
* `.env.template` documents required env vars; copy to `.env` for local dev.
* Features (frontend, keydb, langfuse, qdrant) must be enabled in Tilt when needed.

**Do not** have Copilot modify Helm/Terraform unless explicitly asked; surface TODOs instead.

---

## 8) Anti-patterns (forbidden)

* Business logic in FastAPI routers or Vue components
* Relative imports; wildcard imports; raw SQL with f-strings
* `requests` in async code; `time.sleep()` in app/tests (use `asyncio.sleep`)
* Logging secrets/PII; exposing stack traces to clients

---

## 9) Examples Copilot should imitate


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stackitcloud/rag-template](https://github.com/stackitcloud/rag-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
