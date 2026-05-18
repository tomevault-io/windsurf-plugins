---
trigger: always_on
description: Scope: repo-level guardrails for PlanExe services and shared packages.
---

# PlanExe agent instructions

Scope: repo-level guardrails for PlanExe services and shared packages.
Always check the package-level `AGENTS.md` for file-specific rules
(use `rg --files -g 'AGENTS.md'` if you are unsure).

## Repo architecture map
- `database_api`: shared SQLAlchemy models for DB-backed services.
- `worker_plan/worker_plan_api`: shared API types/helpers (must stay lightweight).
- `worker_plan`: FastAPI service that runs the pipeline.
- `frontend_multi_user`: Flask UI (hosted mode) + Postgres.
- `worker_plan_database`: DB-backed worker that polls tasks.
- `mcp_cloud`: MCP stdio server + HTTP wrapper; primary cloud deployment, secondary Docker setup for advanced users, tertiary venv workflow for developers; bridges MCP tools to PlanExe DB/worker_plan.

## Shared contracts
- Keep `worker_plan` HTTP endpoints and response shapes backward compatible.
- Preserve shared SQLAlchemy models in `database_api` (nullable defaults for new columns).
- Run directory naming defaults live in `worker_plan/worker_plan_api/generate_run_id.py`.
  Verify in code before changing run-id formats.
- Keep prompt catalog UUIDs stable when used as defaults; they live in
  `worker_plan/worker_plan_api/prompt/data/*.jsonl`.

## Hard rules (agent safety)
- **Never commit directly to `main`.** Always create a feature branch
  (e.g. `feat/short-description`, `fix/short-description`) and commit there.
  Push the branch and open a PR so CI can verify the changes.
- Do not add real API keys or passwords to `.env`, `.env.*`, or any `llm_config/*.json` file.
- Treat `track_activity.jsonl` as sensitive (may contain API keys/tokens).
  Never expose it to end users.
  Store `track_activity.jsonl` in `PlanItem.run_track_activity_jsonl` and keep
  it out of downloadable zips at artifact creation time. Legacy snapshots may
  be sanitized at download time, but new snapshots should be served directly
  without unzip/recompress.
- Shared packages (`database_api`, `worker_plan_api`) must not import service apps
  (`frontend_*`, `worker_plan_database`, `worker_plan.app`).
- If a service needs shared logic, move it into a shared package rather than
  importing across service boundaries.

## Environment variables
- Canonical env keys live in `.env.docker-example`, `.env.developer-example`,
  and `worker_plan/worker_plan_api/planexe_dotenv.py`. Read those before adding
  or renaming env vars.

## Cross-service conventions
- `.env` and the `llm_config/` directory are expected in the repo root for Docker setups.
- Profile file mapping:
  - `baseline` -> `llm_config/baseline.json`
  - `premium` -> `llm_config/premium.json`
  - `frontier` -> `llm_config/frontier.json`
  - `custom` -> `llm_config/custom.json` (or `PLANEXE_LLM_CONFIG_CUSTOM_FILENAME`)
- `PLANEXE_MODEL_PROFILE` selects which profile file is used at runtime.
- Baseline is the default profile and fallback if another selected profile file is missing.
- Use `PLANEXE_*` env vars; prefer existing defaults when adding new ones.
- Do not assume ports; read `docker-compose.yml` or the service env defaults
  (`PLANEXE_*_PORT`) before any manual verification.

## Docker notes
- `PLANEXE_POSTGRES_PORT` changes the host port mapping only; containers still
  connect to Postgres on 5432.

## Documentation sync
- When changing Docker services, env defaults, or port mappings, update
  `docker-compose.yml`, `docker-compose.md`, and `docs/docker.md` together.
- When changing quickstart or LLM env requirements (e.g.
  `OPENROUTER_API_KEY`), update `docs/getting_started.md`.
- When changing local dev startup steps or the test command, update
  `docs/install_developer.md`.
- For README links, prefer absolute `https://docs.planexe.org/...` URLs so GitHub
  readers land on the published docs site.

## Testing strategy
- Prefer unit tests over manual curl/server checks.
- Run `python test.py` from repo root for existing coverage.
- If you change logic without tests, add a unit test close to the code.
- Do not start services or run curl checks unless explicitly requested.

## Coding standards
- Type hints: add to all public function signatures.
- Async/sync: FastAPI code can be `async`; Flask routes must stay sync.
- Error handling: do not use bare `except:`; log stack traces.
- Logging: use `logging` (avoid `print()` in service code).
- Formatting/linting: no repo-wide formatter is configured; follow existing
  file style and do not add new lint tools unless requested.

## LLM structured-output schemas (Literal vs Enum)
- In Pydantic models used for LLM structured output, use `Literal["a", "b"]`
  instead of an `Enum` type for field annotations.
- Reason: `Enum` fields cause `model_json_schema()` to emit `$defs`/`$ref`
  indirection. Some structured-output backends (e.g. MLX Outlines for local
  inference) cannot resolve `$defs`/`$ref` and fail silently or error out.
  `Literal` produces a flat schema with an inline `enum` array, which is
  universally supported.
- Keep the canonical `str(Enum)` class in the same module as the single source
  of truth for valid values; the `Literal` duplicates those values in the
  schema annotation only.
- A CI parity test (`test_enum_literal_parity.py`) asserts that every `Literal`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlanExeOrg/PlanExe](https://github.com/PlanExeOrg/PlanExe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
