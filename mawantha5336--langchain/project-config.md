---
trigger: always_on
description: This document provides context to understand the LangChain Python project and assist with development.
---

# Global development guidelines for the LangChain monorepo

This document provides context to understand the LangChain Python project and assist with development.

## Project architecture and context

### Monorepo structure

This is a Python monorepo with multiple independently versioned packages that use `uv`.

```txt
langchain/
├── libs/
│   ├── core/             # `langchain-core` primitives and base abstractions
│   ├── langchain/        # `langchain-classic` (legacy, no new features)
│   ├── langchain_v1/     # Actively maintained `langchain` package
│   ├── partners/         # Third-party integrations
│   │   ├── openai/       # OpenAI models and embeddings
│   │   ├── anthropic/    # Anthropic (Claude) integration
│   │   ├── ollama/       # Local model support
│   │   └── ... (other integrations maintained by the LangChain team)
│   ├── text-splitters/   # Document chunking utilities
│   ├── standard-tests/   # Shared test suite for integrations
│   ├── model-profiles/   # Model configuration profiles
├── .github/              # CI/CD workflows and templates
├── .vscode/              # VSCode IDE standard settings and recommended extensions
└── README.md             # Information about LangChain
```

- **Core layer** (`langchain-core`): Base abstractions, interfaces, and protocols. Users should not need to know about this layer directly.
- **Implementation layer** (`langchain`): Concrete implementations and high-level public utilities
- **Integration layer** (`partners/`): Third-party service integrations. Note that this monorepo is not exhaustive of all LangChain integrations; some are maintained in separate repos, such as `langchain-ai/langchain-google` and `langchain-ai/langchain-aws`. Usually these repos are cloned at the same level as this monorepo, so if needed, you can refer to their code directly by navigating to `../langchain-google/` from this monorepo.
- **Testing layer** (`standard-tests/`): Standardized integration tests for partner integrations

### Development tools & commands

- `uv` – Fast Python package installer and resolver (replaces pip/poetry)
- `make` – Task runner for common development commands. Feel free to look at the `Makefile` for available commands and usage patterns.
- `ruff` – Fast Python linter and formatter
- `mypy` – Static type checking
- `pytest` – Testing framework

This monorepo uses `uv` for dependency management. Local development uses editable installs: `[tool.uv.sources]`

Each package in `libs/` has its own `pyproject.toml` and `uv.lock`.

Before running your tests, set up all packages by running:

```bash
# For all groups
uv sync --all-groups

# or, to install a specific group only:
uv sync --group test
```

```bash
# Run unit tests (no network)
make test

# Run specific test file
uv run --group test pytest tests/unit_tests/test_specific.py
```

```bash
# Lint code
make lint

# Format code
make format

# Type checking
uv run --group lint mypy .
```

#### Environment and dependency management

Use `uv` for all environment and dependency operations in this monorepo. Do not invoke `pip`, `poetry`, or `conda` directly.

- Let `uv` manage the interpreter and virtual environments — `uv sync` and `uv run` operate without manual `source .venv/bin/activate`. Do not create ad-hoc virtual environments outside the package directory.
- Each package targets its own supported Python range via its `pyproject.toml`; do not pin a global Python version. If you need an interpreter explicitly, defer to the package's `requires-python` rather than assuming system Python.
- Install dependencies explicitly through `uv sync` (optionally `--group <name>` / `--all-groups`); never let them install implicitly.
- Don't mix environments within a session, and don't add new dependencies unless strictly required — when you do, justify them (recent releases/commits, adoption).

#### Key config files

- pyproject.toml: Main workspace configuration with dependency groups
- uv.lock: Locked dependencies for reproducible builds
- Makefile: Development tasks

#### PR and commit titles

Follow Conventional Commits. See `.github/workflows/pr_lint.yml` for allowed types and scopes. All titles must include a scope with no exceptions — even for the main `langchain` package.

- Start the text after `type(scope):` with a lowercase letter, unless the first word is a proper noun (e.g. `Azure`, `GitHub`, `OpenAI`) or a named entity (class, function, method, parameter, or variable name).
- Wrap named entities in backticks so they render as code. Proper nouns are left unadorned.
- Keep titles short and descriptive — save detail for the body.

Examples:

```txt
feat(langchain): add new chat completion feature
fix(core): resolve type hinting issue in vector store
chore(anthropic): update infrastructure dependencies
feat(langchain): `ls_agent_type` tag on `create_agent` calls
fix(openai): infer Azure chat profiles from model name
```

#### Branch naming

Branches should be prefixed `<github-username>/<scope>/<short-description>`:

- `<github-username>` — the author's GitHub login (e.g. `mdrxy`).
- `<scope>` — the same scope used in the Conventional Commit title (`core`, `langchain`, partner name, `infra`, `docs`, etc.).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mawantha5336/langchain](https://github.com/Mawantha5336/langchain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
