---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**StatGPT** is an AI-driven Talk-To-Your-Data platform that enables users to interact with official statistics data using natural language. It leverages LLMs to provide relevant data from statistical databases through conversational interfaces.

### Key Capabilities
- Natural language querying of SDMX datasets
- Wide indicator search (semantic + keyword + LLM reasoning)
- Data grounding with hallucination prevention
- Multi-language support for queries and responses
- Automated data visualization with Plotly
- Glossary of terms for consistent terminology

## Version Control

GitHub is used for version control.

Current main branch: `development`

## Commands

### Code Quality
```bash
make format              # Format code (autoflake, black, isort)
make lint                # Run all linters (flake8, black, isort, autoflake, mypy)
make mypy                # Run only mypy type checking
```

### Testing
```bash
make test                # Run all tests (unit + integration)
make test_unit           # Run unit tests only
make test_integration    # Run integration tests (requires test DB containers)
make test_db_migrate     # Run migrations on test database
```

### Database Operations
```bash
make db_migrate          # Apply alembic migrations
make db_downgrade        # Rollback last migration
make db_autogenerate MESSAGE="Your migration message"  # Generate new migration
```

### Localization
```bash
make extract_messages    # Extract translatable strings from formatters
make update_messages     # Update .po files from template
make compile_messages    # Compile .po files to .mo files
make locales             # Shorthand for compile_messages
```

### Virtual Environment
```bash
make install_dev         # Install dev dependencies
make install_all         # Install dev + experiments dependencies
make remove_venv         # Remove and recreate venv
```

### CLI (Interactive Command-Line Interface)
```bash
make statgpt_cli        # Start the StatGPT CLI
```

CLI uses `STATGPT_CLI_*` prefixed environment variables.
See `statgpt/cli/README.md` for full documentation.

**Available Commands:**
| Command | Description |
|---------|-------------|
| `auth login` | Authenticate with the admin API |
| `auth logout` | Clear cached authentication token |
| `auth status` | Show current authentication status |
| `channel list` | List all available channels |
| `channel import` | Import channel from zip archive |
| `channel status` | Show dataset preprocessing status |
| `channel deduplicate` | Deduplicate embeddings for a channel |
| `channel reindex` | Reindex dataset embeddings for a channel |
| `content init` | Initialize channels, data sources, datasets, glossaries |
| `settings` | Show current CLI settings |

## Architecture Overview

### Project Structure

```
statgpt/
├── app/                 # Chat Backend (DIAL Application)
│   ├── application/     # App factory and DIAL app setup
│   ├── chains/          # LangChain orchestration and agent tools
│   │   ├── data_query/  # SDMX data query tool
│   │   ├── file_rags/   # Publications RAG tool
│   │   ├── web_search/  # Web search tool
│   │   ├── datasets_meta/       # Available datasets tool
│   │   ├── glossary_tools.py    # Glossary tools
│   │   └── supreme_agent.py     # Main agent orchestrator
│   ├── schemas/         # Pydantic models
│   ├── services/        # Business logic
│   ├── settings/        # Pydantic Settings configuration
│   └── utils/           # Utilities (formatters with i18n)
├── admin/               # Admin Backend (FastAPI standalone)
│   ├── routers/         # API routes (channels, datasets, data sources, glossary)
│   ├── services/        # Admin business logic
│   ├── auth/            # OIDC authentication
│   ├── alembic/         # Database migrations
│   └── settings/        # Admin configuration
├── common/              # Shared code
│   ├── models/          # SQLAlchemy database models
│   ├── data/            # Data access layer and SDMX handling
│   │   ├── sdmx/        # SDMX protocol implementation (v2.1)
│   │   └── quanthub/    # QuantHub SDMX provider
│   ├── vectorstore/     # PGVector storage implementation
│   ├── hybrid_indexer/  # Vector indexing for semantic search
│   ├── services/        # Shared services
│   └── schemas/         # Shared Pydantic models
└── cli/                 # Interactive command-line interface
    ├── commands/        # Command implementations (auth, channel, content, settings)
    └── shared/          # Shared utilities (admin client, console, prompts, settings)
        └── auth/        # Pluggable auth providers (azure, keycloak)

tests/
├── unit/                # Unit tests (no external dependencies)
└── integration/         # Integration tests (requires test DB containers)
```

### Agent Architecture

StatGPT uses a **tool-calling agent** approach:
- Main agent (`supreme_agent.py`) orchestrates all tools
- History consists of **static** (system prompt, predefined calls) and **dynamic** (user queries, tool calls, responses) blocks
- All data responses are grounded in actual query results to prevent hallucinations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epam/statgpt-backend](https://github.com/epam/statgpt-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
