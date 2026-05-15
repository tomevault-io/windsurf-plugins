---
trigger: always_on
description: This repository contains SDKs and CLI for invoking Metadata Dynamic Agents.
---

# CLAUDE.md - Metadata AI SDK

This repository contains SDKs and CLI for invoking Metadata Dynamic Agents.

## Repository Structure

```
metadata-ai-sdk/
├── cli/                    # Rust CLI (ai-sdk)
├── python/                 # Python SDK (ai-sdk on PyPI)
│   ├── src/ai_sdk/    # Source code
│   ├── tests/              # Unit and integration tests
│   └── pyproject.toml      # Package config
├── typescript/             # TypeScript SDK (@openmetadata/ai-sdk on npm)
├── java/                   # Java SDK (org.open-metadata:ai-sdk)
├── n8n-nodes-metadata/     # n8n community node (depends on TypeScript SDK)
├── cookbook/                # Guides, scripts, and resources for common use cases
│   └── resources/
│       └── demo-database/  # Jaffle Shop demo (PostgreSQL + Metabase + dbt)
│           ├── docker/     # docker-compose.yml, Dockerfile, init.sql
│           └── dbt/        # dbt project (models, profiles, tests)
├── scripts/                # Development scripts (pre-commit hooks)
├── .claude/skills/         # Claude Code skills for this repo
├── VERSION                 # Single source of truth for version
└── Makefile                # Cross-SDK commands (single source of truth)
```

## Quick Commands

```bash
# Linting & Formatting
make lint              # Check all SDKs (CI equivalent)
make format            # Fix all SDKs (pre-commit equivalent)

# Testing
make test-all          # Run all unit tests
make test-integration  # Run integration tests (requires AI_SDK_HOST, AI_SDK_TOKEN)

# Building
make build-all         # Build all SDKs

# Version Management
make version           # Show current version
make bump-version V=X.Y.Z  # Update all SDKs

# Local Development
make install-local       # Install Python SDK in editable mode (all extras)

# Demo Database
make install-dbt         # Install dbt-postgres for the demo
make demo-database       # Start PostgreSQL (port 5433) + Metabase (port 3000)
make demo-database-stop  # Stop demo database containers
make demo-dbt            # Run dbt models and tests against the demo database
make demo-gdpr           # Start the GDPR DSAR compliance demo (port 8080)

# Git Hooks
make install-hooks     # Install pre-commit hooks
```

## Architecture

### Design Principles

1. **Multi-SDK Consistency** - All SDKs implement the same API surface with language-idiomatic patterns
2. **Zero/Minimal Dependencies** - TypeScript has zero runtime deps; others minimize external deps
3. **Streaming First** - SSE streaming is a primary use case, not an afterthought
4. **Typed Errors** - Structured error hierarchy for proper error handling

### API Pattern (All SDKs)

```
client = AISdk(host, token)

# default AskCollate agent
response = client.agent().invoke("message")

# Named dynamic agent
response = client.agent("agent-name").invoke("message")

# Streaming (works with both)
for event in client.agent().stream("message"):
    handle(event)

# Entity CRUD lives on namespaces (composition pattern)
client.agents.list()
client.bots.get("ingestion-bot")
client.personas.list()
client.abilities.get("DataQuality")

# Context Center memories
client.memories.list(primary_entity_fqn="db.tbl")
client.memories.create(CreateContextMemoryRequest(name="m1", question="...", answer="..."))
results = client.memories.search("explain customer churn")
```

### Event Types (Streaming)
- `message` - Partial response content
- `tool_call` - Agent calling a tool
- `tool_result` - Tool result
- `error` - Error occurred
- `done` - Stream complete

### Error Hierarchy
```
AISdkError (base)
├── AuthenticationError (401)
├── AuthorizationError (403)
├── NotFoundError (404)
├── ValidationError (400)
├── RateLimitError (429)
└── ServerError (5xx)
```

## SDK Development

### Naming Conventions

| Concept | Python | TypeScript | Java | Rust |
|---------|--------|------------|------|------|
| Methods | `snake_case` | `camelCase` | `camelCase` | `snake_case` |
| Classes | `PascalCase` | `PascalCase` | `PascalCase` | `PascalCase` |
| Files | `snake_case.py` | `camelCase.ts` | `PascalCase.java` | `snake_case.rs` |

### Adding a New Method

When adding functionality, update ALL SDKs:

1. **Python** (`python/src/ai_sdk/`)
   - Add to `_client.py` or `_agent.py`
   - Add types to models
   - Add tests in `python/tests/`

2. **TypeScript** (`typescript/src/`)
   - Add to `agent.ts`
   - Add interfaces to `models.ts`
   - Add tests in `typescript/tests/`

3. **Java** (`java/src/main/java/io/metadata/ai/`)
   - Add to `AgentHandle.java`
   - Add model classes in `models/`
   - Add tests in `java/src/test/`

4. **Rust CLI** (`cli/src/`)
   - Add subcommand in `main.rs`
   - Add tests

5. **Verify**
   ```bash
   make lint && make test-all
   ```

### Testing

**Unit Tests** - Mock HTTP responses, no real API calls:
```bash
cd python && pytest --ignore=tests/integration
cd typescript && npm test
cd java && mvn test -DexcludedGroups=integration
cd cli && cargo test --lib
```

**Integration Tests** - Real API calls (requires credentials):
```bash
export AI_SDK_HOST=https://your-instance.getcollate.io
export AI_SDK_TOKEN=your-jwt-token
export AI_SDK_TEST_AGENT=agent-name  # optional

make test-integration
```

## Code Quality

### Linting Tools

| SDK | Formatter | Linter | Type Checker |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-metadata/ai-sdk](https://github.com/open-metadata/ai-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
