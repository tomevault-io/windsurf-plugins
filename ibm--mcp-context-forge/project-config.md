---
trigger: always_on
description: Guidelines for AI coding assistants working with this repository.
---

# AGENTS.md

Guidelines for AI coding assistants working with this repository.

For domain-specific guidance, see subdirectory AGENTS.md files:
- `tests/AGENTS.md` - Testing conventions and workflows
- `plugins/AGENTS.md` - Plugin framework and development
- `charts/AGENTS.md` - Helm chart operations
- `docs/AGENTS.md` - Documentation authoring
- `mcp-servers/AGENTS.md` - MCP server implementation
- `crates/mcp_runtime/DEVELOPING.md` - Rust MCP runtime development workflows, command matrix, and validation

**Note:** The `llms/` directory contains guidance for LLMs *using* ContextForge solution (end-user runtime guidance), not for code agents working on this codebase.

## Project Overview

ContextForge is an open source registry and proxy that federates MCP, A2A, and REST/gRPC APIs with centralized governance, discovery, and observability. It federates tools, agents, and APIs, optimizes agent and tool calling, and supports plugins, auth/RBAC, rate-limiting, virtual servers, multi-transport protocols, and an optional Admin UI.

## Project Structure

```
mcpgateway/                 # Core FastAPI application
├── main.py                 # Application entry point
├── config.py               # Environment configuration
├── db.py                   # SQLAlchemy ORM models and session management
├── schemas.py              # Pydantic validation schemas
├── services/               # Business logic layer (50+ services)
├── routers/                # HTTP endpoint definitions (19 routers)
├── middleware/             # Cross-cutting concerns (16 middleware)
├── transports/             # Protocol implementations (SSE, WebSocket, stdio, streamable HTTP)
├── plugins/                # Plugin framework infrastructure
└── alembic/                # Database migrations

tests/                      # Test suite (see tests/AGENTS.md)
plugins/                    # Plugin implementations (see plugins/AGENTS.md)
plugin_templates/           # Starter templates for building new plugins
charts/                     # Helm charts (see charts/AGENTS.md)
docs/                       # Architecture and usage documentation (see docs/AGENTS.md)
a2a-agents/                 # A2A agent implementations (used for testing/examples)
mcp-servers/                # MCP server templates (see mcp-servers/AGENTS.md)
crates/                     # Direct Rust crate folders (runtime and wrapper)
llms/                       # End-user LLM guidance (not for code agents)
```

## Essential Commands

### Setup
```bash
cp .env.example .env && make install-dev check-env    # Complete setup
make venv                          # Create virtual environment with uv
make install-dev                   # Install with dev dependencies (includes build-ui)
make check-env                     # Verify .env against .env.example
make build-ui                      # Rebuild Admin UI JS bundle (requires npm)
```

### Development
```bash
make dev                          # Dev server on :8000 with autoreload
make serve                        # Production gunicorn on :4444
make serve-ssl                    # HTTPS on :4444 (creates certs if needed)
```

### Code Quality
```bash
# After writing code
make autoflake isort black pre-commit

# Before committing, use ty, mypy and pyrefly to check just the new files, then run:
make ruff bandit interrogate pylint verify

# Before committing Rust changes (tools_rust/):
# Run fmt-check, clippy -D warnings, and cargo test for Rust crates
cd tools_rust/mcp_runtime && cargo fmt --check && cargo clippy -- -D warnings && cargo test
```

## Authentication & RBAC Overview

ContextForge implements a **two-layer security model**:

1. **Token Scoping (Layer 1)**: Controls what resources a user CAN SEE (data filtering)
2. **RBAC (Layer 2)**: Controls what actions a user CAN DO (permission checks)

### Token Scoping Quick Reference

**API / legacy tokens** — JWT `teams` claim is the sole authority (`normalize_token_teams()`):

| JWT `teams` State | `is_admin: true` | `is_admin: false` |
|-------------------|------------------|-------------------|
| Key MISSING | PUBLIC-ONLY `[]` | PUBLIC-ONLY `[]` |
| `teams: null` | ADMIN BYPASS | PUBLIC-ONLY `[]` |
| `teams: []` | PUBLIC-ONLY `[]` | PUBLIC-ONLY `[]` |
| `teams: ["t1"]` | Team + Public | Team + Public |

**Session tokens** (`token_use: "session"`) — DB is the authority; JWT `teams` only narrows (`resolve_session_teams()`):

| JWT `teams` State | DB admin? | Result | Access Level |
|-------------------|-----------|--------|--------------|
| any | yes | `None` | ADMIN BYPASS (DB authority) |
| Missing/null/`[]` | no | DB teams | Full DB membership |
| `["t1"]` | no | intersection | Narrowed to overlap |
| `["revoked"]` | no | `[]` | Public-only (fail-closed) |

**Key behaviors:**

- **API/legacy tokens**: Missing `teams` key = public-only access (secure default). Admin bypass requires BOTH `teams: null` AND `is_admin: true`. `normalize_token_teams()` in `mcpgateway/auth.py` is the single source of truth.
- **Session tokens**: Admin bypass is determined by the DB `is_admin` flag, not the JWT `teams` claim. Non-admin sessions can be narrowed via JWT `teams`. `resolve_session_teams()` in `mcpgateway/auth.py` is the single policy point.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
