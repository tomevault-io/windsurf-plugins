---
trigger: always_on
description: Repo operating notes and ground truth for coding agents working on MUTX.
---


# AGENTS.md

Repo guidance for agentic coding agents working in `/Users/fortune/MUTX`.

## Rule Files

* No `.cursorrules` file exists.
* No files exist under `.cursor/rules/`.
* No `.github/copilot-instructions.md` file exists.

## Repo Map

* `app/`: Next.js 16 App Router frontend.
* `app/api/`: Next.js route handlers that proxy or handle website requests.
* `components/`: shared React UI components.
* `lib/`: shared TypeScript utilities, docs helpers, desktop release helpers, and Pico support modules.
* `src/api/`: FastAPI backend, models, services, middleware, and integrations.
* `cli/`: Click-based Python CLI.
* `sdk/mutx/`: Python SDK package.
* `tests/api/`: pytest API tests.
* `tests/unit/`: Jest frontend unit tests.
* `tests/website.spec.ts`: Playwright smoke tests.
* `infrastructure/`: Terraform, Ansible, monitoring, and related Make targets.
* `infrastructure/helm/`: Kubernetes Helm chart for MUTX deployment.

## Ground Truth And Known Drift

* Trust source code and config over README/docs when they disagree.
* FastAPI public control-plane routes are mounted under `/v1/*`, with root probes at `/`, `/health`, `/ready`, and `/metrics`.
* RBAC is now enforced on all routes — roles are checked via dependencies in `src/api/routes/`. See `src/api/security.py` for the enforcement layer.
* OIDC token validation exists at `src/api/auth/oidc.py`; configure via `OIDC_ISSUER`, `OIDC_CLIENT_ID`, and `OIDC_JWKS_URI` environment variables.
* `docs/api/agents.md` is current: `POST /agents` ownership comes from the bearer token (no `user_id` in request body); auth dependencies are attached.
* Parts of older docs still drift; check `src/api/main.py`, `src/api/routes/`, and `docs/api/openapi.json` before copying examples.
* The repo uses flat-config ESLint via `eslint.config.mjs`; `npm run lint` now checks the repo surface with `eslint . --max-warnings=0`.
* `npm run build` uses plain `next build`.
* Recent repo-level fixes covered `/v1/auth/*` docs drift, the email verification expiry migration backfill guard, the CLI/SDK distribution-name split, and the stale versioning unit export name.
* Playwright runs against the local standalone server from `playwright.config.ts`; build before e2e runs so `.next/standalone` exists.
* `scripts/test.sh` is still the best repo-native baseline, but `package.json` now exposes `npm test` and `npm run typecheck` as explicit frontend entrypoints.

## Setup And Environment

* CI uses Node 20 for frontend checks.
* CI uses Python 3.11 for backend checks; root package requires `>=3.10`, SDK declares `>=3.9`.
* Common bootstrap:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e ".[dev]"
npm install
cp .env.example .env
```

* Full local stack: `./dev.sh`
* Backend only: `uvicorn src.api.main:app --reload --port 8000`
* Frontend only: `npm run dev`

## Build, Lint, And Test Commands

**Frontend**

```bash
npm run dev
npm run build
npm run generate-types
```

* `npm run build` performs a production build.
* `npm run typecheck` is the explicit frontend TypeScript gate.
* `npm run lint` is repo-wide via `eslint . --max-warnings=0`.

**Python checks**

```bash
ruff check src/api cli sdk
ruff check src/api/routes/agents.py
black --check src/api cli sdk
black src/api/routes/agents.py
python -m compileall src/api cli sdk/mutx
```

* If `ruff` or `black` is missing, install dev extras with `pip install -e ".[dev]"`.

**Pytest API tests**

```bash
./.venv/bin/python -m pytest --collect-only -q
./.venv/bin/python -m pytest
./.venv/bin/python -m pytest tests/api/test_agents.py
./.venv/bin/python -m pytest tests/api/test_agents.py::TestCreateAgent::test_create_agent_success -q
```

* Single-test shape: `path/to/test_file.py::TestClass::test_name`.
* Current reality: collection works; when pytest goes red, check migration head drift and auth docs drift first.

**Playwright smoke tests**

```bash
npx playwright test
npx playwright test tests/website.spec.ts
npx playwright test tests/website.spec.ts:4
npx playwright test -g "no console errors"
npx playwright test --list
```

* Playwright targets the local standalone app server configured in `playwright.config.ts`.
* Build first if `.next/standalone/server.js` is missing.

**Infrastructure**

```bash
make -C infrastructure tf-fmt
make -C infrastructure tf-validate
make -C infrastructure tf-plan-staging
make -C infrastructure ansible-lint
make -C infrastructure monitor-validate
```

* Direct Terraform also works, for example `terraform -chdir=infrastructure/terraform plan ...`.

## Observability

OpenTelemetry is used for distributed tracing across the MUTX platform.

**Span Naming Convention**: All spans should follow the pattern `mutx.<operation>`:
- `mutx.agent.execute` - Agent execution
- `mutx.agent.initialize` - Agent initialization
- `mutx.llm.call` - LLM API calls
- `mutx.tool.execution` - Tool execution
- `mutx.session.start` - Session start
- `mutx.session.end` - Session end

**Standard Attributes**:
- `agent.id` - Unique identifier for the agent
- `session.id` - Session identifier for the interaction
- `trace.id` - Distributed trace identifier

**SDK Usage** (`sdk/mutx/telemetry.py`):
```python
from mutx.telemetry import init_telemetry, span, trace_context, get_tracer


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mutx-dev/mutx-dev](https://github.com/mutx-dev/mutx-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
