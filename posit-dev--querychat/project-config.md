---
trigger: always_on
description: querychat is a multilingual package that allows users to chat with their data using natural language queries. It's available for R (Shiny) and Python (Shiny for Python).
---

# querychat

## Project Overview

querychat is a multilingual package that allows users to chat with their data using natural language queries. It's available for R (Shiny) and Python (Shiny for Python).

The core functionality translates natural language queries into SQL statements that are executed against data sources. This approach ensures reliability, transparency, and reproducibility by:

1. Leveraging LLMs' strengths in writing SQL
2. Providing transparency with visible SQL queries
3. Enabling reproducibility through reusable queries

## Repository Structure

The repository contains separate packages for R and Python:

```
/
├── pkg-r/                  # R package implementation
│   ├── R/                  # R source files (R6 classes and utilities)
│   │   ├── QueryChat.R             # Main QueryChat R6 class
│   │   ├── DataSource.R            # Abstract DataSource base class
│   │   ├── DataFrameSource.R       # DataSource for data.frames
│   │   ├── DBISource.R             # DataSource for DBI connections
│   │   ├── TblSqlSource.R          # DataSource for dbplyr tbl_sql
│   │   ├── QueryChatSystemPrompt.R # System prompt management (internal)
│   │   ├── querychat_module.R      # Shiny module functions (internal)
│   │   ├── querychat_tools.R       # Tool definitions for LLM
│   │   ├── deprecated.R            # Deprecated functional API
│   │   └── utils-*.R               # Utility functions
│   ├── inst/               # Installed files
│   │   ├── examples-shiny/ # Shiny example applications
│   │   ├── htmldep/        # HTML dependencies
│   │   └── prompts/        # Prompt templates
│   └── tests/              # testthat test suite
│
├── pkg-py/       # Python package implementation
│   ├── src/      # Python source files
│   ├── tests/    # pytest test suite
│   └── examples/ # Example applications
│
├── docs/ # Documentation site
├── _dev/ # Development utilities and demos (local scratch space only)
```

## Common Commands

### Python Package

We use `uv` for Python package management and `make` for common tasks.

```bash
# Setup Python environment
make py-setup

# Run Python checks (format, types, tests)
make py-check
make py-check-format
make py-check-types
make py-check-tests

# Format Python code
make py-format

# Build Python package
make py-build

# Build Python documentation
make py-docs
```

Before committing any Python code, you must run all three checks and confirm they pass:

```bash
uv run ruff check --fix pkg-py --config pyproject.toml
make py-check-types
make py-check-tests
```

Do not commit or push until all three pass.

### R Package

```bash
# Install R dependencies
make r-setup

# Run R checks (format, tests, package)
make r-check
make r-check-format
make r-check-tests
make r-check-package

# Format R code
make r-format

# Document R package
make r-document

# Build R documentation
make r-docs
```

### Documentation

```bash
# Build all documentation
make docs

# Preview R docs
make r-docs-preview

# Preview Python docs
make py-docs-preview
```

## Code Architecture

### Core Components

Both R and Python implementations use an object-oriented architecture:

1. **Data Sources**: Abstractions for data frames and database connections that provide schema information and execute SQL queries
   - R: R6 class hierarchy in `pkg-r/R/`
     - `DataSource` - Abstract base class defining the interface (`DataSource.R`)
     - `DataFrameSource` - For data.frame objects (`DataFrameSource.R`)
     - `DBISource` - For DBI database connections (`DBISource.R`)
     - `TblSqlSource` - For dbplyr tbl_sql objects (`TblSqlSource.R`)
   - Python: `DataSource` classes in `pkg-py/src/querychat/datasource.py`

2. **LLM Client**: Integration with LLM providers (OpenAI, Anthropic, etc.) through:
   - R: ellmer package
   - Python: chatlas package

3. **Query Chat Interface**: Main orchestration class that manages the chat experience:
   - R: `QueryChat` R6 class in `pkg-r/R/QueryChat.R`
     - Provides methods: `$new()`, `$app()`, `$sidebar()`, `$ui()`, `$server()`, `$df()`, `$sql()`, etc.
     - Internal Shiny module functions: `mod_ui()` and `mod_server()` in `pkg-r/R/querychat_module.R`
   - Python: `QueryChat` class in `pkg-py/src/querychat/querychat.py`

4. **System Prompt Management**:
   - R: `QueryChatSystemPrompt` R6 class in `pkg-r/R/QueryChatSystemPrompt.R`
     - Handles loading and rendering of prompt templates with Mustache
     - Manages data descriptions and extra instructions
   - Python: Similar logic in `QueryChat` class

5. **Prompt Engineering**: System prompts and tool definitions that guide the LLM:
   - R: `pkg-r/inst/prompts/`
     - Main prompt (`prompt.md`)
     - Tool descriptions (`tool-query.md`, `tool-reset-dashboard.md`, `tool-update-dashboard.md`)
   - Python: `pkg-py/src/querychat/prompts/`
     - Main prompt (`prompt.md`)
     - Tool descriptions (`tool-query.md`, `tool-reset-dashboard.md`, `tool-update-dashboard.md`)

### R Package Architecture

The R package uses R6 classes for object-oriented design:

- **QueryChat**: Main user-facing class that orchestrates the entire query chat experience
  - Takes data sources as input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/querychat](https://github.com/posit-dev/querychat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
