---
trigger: always_on
description: **Project**: Library-first AI conversation export parser with multi-provider support
---

# echomine Development Guidelines

**Project**: Library-first AI conversation export parser with multi-provider support
**Primary Use Case**: cognivault integration + standalone CLI for researchers
**Last Updated**: 2025-11-22

---

## Active Technologies
- Python 3.12+ (existing stack, mypy --strict compliant) (002-advanced-search)
- File system only (JSON exports, no database) (002-advanced-search)
- Python 3.12+ (existing stack, mypy --strict compliant) + Pydantic v2.6+, ijson 3.2+, typer 0.9+, rich 13.0+, structlog 23.0+ (003-baseline-enhancements)

### Core Stack
- **Python 3.12+**: Required for modern type hints (PEP 695, improved generics)
- **Pydantic v2**: Immutable data models with strict validation
- **ijson**: Streaming JSON parser for O(1) memory usage (1GB+ files)
- **typer**: CLI framework with type hints
- **rich**: Terminal output formatting (tables, progress bars, syntax highlighting)
- **structlog**: JSON structured logging with contextual fields

### Development Tools
- **pytest**: Test framework with pytest-cov, pytest-mock, pytest-benchmark
- **mypy**: Static type checker (--strict mode enforced)
- **ruff**: Fast linter and formatter (replaces black, isort, flake8)
- **pre-commit**: Git hooks for quality gates

### Why These Choices
- **ijson over json.load()**: Prevents loading entire 1GB+ file into memory (Constitution Principle VIII)
- **Pydantic over dataclasses**: Comprehensive validation, immutability, JSON schema export
- **typer over click**: Native type hint support, automatic help generation
- **structlog over logging**: JSON output for observability, contextual fields

---

## Project Structure

```
echomine/
├── .claude/
│   ├── agents/              # 11 specialized sub-agents
│   ├── agents.md            # Agent coordination guide (READ THIS)
│   └── commands/            # Spec-kit commands (/speckit.*)
├── specs/
│   └── 001-ai-chat-parser/
│       ├── spec.md          # Feature specification with FRs
│       ├── plan.md          # Implementation plan
│       ├── tasks.md         # Task breakdown
│       └── contracts/       # CLI contract tests
├── src/echomine/
│   ├── models/              # Pydantic models (Message, Conversation, SearchQuery)
│   ├── protocols/           # ConversationProvider protocol
│   ├── adapters/            # Provider implementations (openai/, future: claude/, gemini/)
│   ├── search/              # BM25 ranking algorithms (future)
│   ├── utils/               # Logging, exceptions (NO business logic)
│   └── cli/                 # CLI commands (wraps library, no business logic)
├── tests/
│   ├── unit/                # Fast, isolated tests (70% of test pyramid)
│   ├── integration/         # Component interaction tests (20%)
│   ├── contract/            # FR validation tests (5%)
│   ├── performance/         # pytest-benchmark tests (5%)
│   └── fixtures/            # Test data (sample_export.json, generate_large_export.py)
└── pyproject.toml           # Dependencies, tool configs (mypy, ruff, pytest)
```

**Key Principles**:
- `src/echomine/` = library (importable, reusable)
- `src/echomine/cli/` = thin wrapper over library
- `tests/` mirrors `src/` structure
- `fixtures/` contains reusable test data

---

## Constitution Principles (Non-Negotiable)

All code, architecture, and design decisions MUST comply with these 8 principles:

### I. Library-First Architecture
- ✅ Core functionality in `src/echomine/` as importable library
- ✅ CLI in `src/echomine/cli/` wraps library (NEVER the reverse)
- ✅ All features available programmatically (cognivault integration use case)
- ❌ NEVER put business logic in CLI commands

### II. CLI Interface Contract
- ✅ Results to **stdout** (JSON via --json, human-readable by default)
- ✅ Progress and errors to **stderr**
- ✅ Exit codes: 0 (success), 1 (operational error), 2 (usage error)
- ✅ Pipeline-friendly (composable with jq, xargs, grep)

### III. Test-Driven Development (TDD)
- ✅ **RED**: Write failing test FIRST (verify it fails)
- ✅ **GREEN**: Write minimal code to pass test
- ✅ **REFACTOR**: Improve code while keeping tests green
- ❌ NEVER write implementation before tests
- ❌ NEVER commit without test coverage

### IV. Observability & Debuggability
- ✅ JSON structured logs via structlog (timestamp, level, operation, message)
- ✅ Contextual fields: file_name, conversation_id, message_id, count
- ✅ Text-based I/O (human-inspectable JSON exports)
- ✅ WARNING logs for skipped malformed entries (graceful degradation)

### V. Simplicity & YAGNI
- ✅ Implement ONLY what spec requires (no speculative features)
- ✅ Simplest solution that meets requirements
- ⚠️ Justify ANY complexity (e.g., ijson required for memory efficiency)
- ❌ No premature abstractions or over-engineering

### VI. Strict Typing Mandatory
- ✅ **ZERO TOLERANCE**: mypy --strict MUST pass (no exceptions)
- ✅ Type hints on ALL functions, methods, variables
- ✅ No `Any` types (use Protocol or TypeVar instead)
- ✅ Pydantic models for ALL structured data
- ✅ Protocol classes for abstractions (runtime_checkable)
- ✅ **Data Integrity**: Model data as it exists in source (nullable fields stay Optional, not hidden with defaults)

### VII. Multi-Provider Adapter Pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aucontraire/echomine](https://github.com/aucontraire/echomine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
