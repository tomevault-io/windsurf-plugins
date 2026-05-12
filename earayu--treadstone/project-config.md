---
trigger: always_on
description: Agent-native sandbox platform for AI agents. Run code, build projects, deploy environments, and hand off browser sessions via CLI, SDK, or REST API. Open source and self-hostable.
---

# Treadstone

Agent-native sandbox platform for AI agents. Run code, build projects, deploy environments, and hand off browser sessions via CLI, SDK, or REST API. Open source and self-hostable.

## Start Here

Use the matching local skill before you act:

| If you need to... | Use |
|-------|-------------|
| Set up this repo for the first time | `dev-setup` |
| Ship code, PRs, merges, releases (GitHub Actions), or prod deploy | `dev-lifecycle` |
| Add or change SQLAlchemy models / Alembic migrations | `database-migration` |
| Answer Neon-specific questions or plan Neon usage | `neon-postgres` |
| Audit a subsystem against the current code and write a detailed report | `system-audit-report` |
| Refresh an existing audit report against the latest code | `audit-report-refresh` |
| Trace runtime architecture and end-to-end data flow | `architecture-data-flow-trace` |
| Plan, write, or validate public docs, docs IA, `llms.txt`, or the docs manifest | [`treadstone-public-docs`](.agents/skills/treadstone-public-docs/SKILL.md) |
| Run, design, or analyze sandbox benchmark / load tests | [`benchmark`](.agents/skills/benchmark/SKILL.md) |

Skills live under `.agents/skills/*/SKILL.md`. AGENTS.md defines repo facts and guardrails; skills define procedures.

## Tech Stack

- Python 3.12+, FastAPI, Uvicorn, SQLAlchemy (async), asyncpg
- Database: Neon (Serverless PostgreSQL)
- Package managers: uv (Python), pnpm (web)
- Lint/format: ruff (Python), ESLint (web)
- Testing: pytest + pytest-asyncio + httpx; Hurl for E2E
- Containerization: Docker
- Orchestration: Kubernetes (Kind for dev, GKE/EKS/AKS for prod)

## Project Structure

```
treadstone/        # Server application source
  main.py          # FastAPI entrypoint
  config.py        # pydantic-settings configuration
  core/            # Database engine, shared utilities
  models/          # SQLAlchemy models
  api/             # API routers
  auth/            # Authentication
  services/        # Business logic
cli/               # CLI package (standalone, published as treadstone-cli)
  treadstone_cli/  # CLI source (click + httpx + rich)
sdk/python/        # Python SDK (published as treadstone-sdk)
tests/             # pytest test suites
  api/             # FastAPI route tests via ASGITransport
  unit/            # Pure logic / model / helper tests
  integration/     # Real DB tests, excluded by default
  e2e/             # Hurl E2E tests (run against deployed cluster)
alembic/           # Database migrations
deploy/            # Helm charts, Kind config, K8s manifests
docs/              # Design docs and plans (zh-CN)
scripts/           # Helper scripts (release, install, deploy, E2E)
.agents/skills/    # AI Agent reusable skills
```

## Skills

| Skill | When to use |
|-------|-------------|
| `dev-setup` | First-time environment setup (once per clone) |
| `dev-lifecycle` | Feature/fix: branch, TDD, ship, PR, merge; release via Actions; agreed codewords (合并代码 / 发版本 / 发生产); see skill |
| `database-migration` | Adding/modifying SQLAlchemy models and Alembic migrations |
| `neon-postgres` | Neon-specific questions (branching, connection methods, SDKs) |
| `system-audit-report` | First-pass or general subsystem audits grounded in the current code |
| `audit-report-refresh` | Re-auditing a subsystem and updating an existing report against the latest code |
| `architecture-data-flow-trace` | Tracing runtime architecture, state transitions, and end-to-end data flow |
| [`treadstone-public-docs`](.agents/skills/treadstone-public-docs/SKILL.md) | Public docs system, docs IA, manifest-driven delivery, `llms.txt` / sitemap / robots, control vs data plane narratives, dual human/agent quality |
| [`benchmark`](.agents/skills/benchmark/SKILL.md) | Run, design, or analyze sandbox benchmark / load tests in `tests/benchmark/` |

For Kubernetes deployment, use **`make local`**, **`make destroy-local`**, and **`make prod`** (context checks before Helm); details in [`deploy/README.md`](deploy/README.md).

## Code Conventions

- All code comments and commit messages in English. Docs default to Chinese in `docs/zh-CN/`.
- **All GitHub-public content must be in English**: commit messages, PR titles/bodies, Issue titles/bodies, review comments, release notes.
- Root-facing docs such as `README.md`, `AGENTS.md`, and `.agents/skills/*/SKILL.md` should stay concise and easy for both humans and agents to scan.
- Async everywhere: all DB operations, HTTP calls, and API handlers must be async.
- TDD: write a failing test first, implement, verify it passes.
- DRY, YAGNI: no premature abstraction.
- All function signatures must have type hints.
- When adding or changing auth, admin, API key, sandbox lifecycle, browser hand-off, or other user-facing control-plane features, update Audit Log coverage and structured request logging in the same change.
- Ruff rules: E, F, I, UP (see `pyproject.toml`). Line width: 120.

## Error Handling

All API errors must return a consistent JSON envelope:

```json
{"error": {"code": "snake_case_code", "message": "Human-readable detail.", "status": 409}}
```

Rules:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [earayu/treadstone](https://github.com/earayu/treadstone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
