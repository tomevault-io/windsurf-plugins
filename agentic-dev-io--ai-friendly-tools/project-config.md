---
trigger: always_on
description: AIFT (AI-Friendly Tools) is a collection of command-line tools optimized for AI interaction. The project consists of multiple Python packages managed as a UV workspace:
---

# Copilot Repository Instructions

## Project Overview

AIFT (AI-Friendly Tools) is a collection of command-line tools optimized for AI interaction. The project consists of multiple Python packages managed as a UV workspace:

- **core**: Core library with config, logging, and CLI framework
- **web**: Web intelligence suite (search, scrape, API tools)
- **mcp-manager**: DuckDB-based MCP manager with semantic search

Note: The `memo` package (Memory & AI tools) exists in the repository but is not currently included in the UV workspace configuration.

## Build and Test Commands

### Setup
```bash
# Install dependencies using UV (recommended)
uv sync --all-groups

# Alternative: Install workspace packages in development mode using pip
# Note: memo package is not in the workspace, install separately if needed
pip install -e core web mcp-manager
pip install -e memo  # Optional, not in workspace
```

### Testing
```bash
# Run all tests
uv run pytest tests/ -v

# Run tests with coverage (for workspace packages)
uv run pytest tests/ -v --cov=core/src --cov=web/src --cov=mcp-manager/src --cov-report=term-missing

# Run specific test file
uv run pytest tests/test_cli.py -v

# Quick smoke test
aift test
```

### Linting and Formatting
```bash
# Check code style with Ruff
uv run ruff check . --select E,F,I,N,W

# Format code with Ruff
uv run ruff format .

# Check formatting without changes
uv run ruff format . --check
```

### Type Checking
```bash
# Run type checker with MyPy (for workspace packages)
uv run mypy core/src web/src mcp-manager/src --strict
```

### Docker
```bash
# Build Docker image
docker buildx build -t aift-os:latest . --load

# Start services
docker-compose up -d

# Access container shell
docker-compose exec aift-os bash

# View logs
docker-compose logs -f
```

## Code Style and Conventions

### Python Standards
- **Python Version**: >=3.11 (as specified in pyproject.toml; CI tests 3.11 and 3.12)
- **Line Length**: 100 characters (configured in pyproject.toml)
- **Import Ordering**: Use Ruff's isort integration (I rules)
- **Naming**: Follow PEP 8 conventions (N rules)

### Code Quality
- Run Ruff linter before committing (`uv run ruff check .`)
- Format code with Ruff (`uv run ruff format .`)
- Type hints are encouraged; MyPy checks are run in CI with `--strict` flag
- All new features should include tests
- Maintain test coverage for modified code

### Testing Guidelines
- Place tests in the `tests/` directory at repository root
- Follow pytest conventions:
  - Test files: `test_*.py`
  - Test classes: `Test*`
  - Test functions: `test_*`
- Use descriptive test names that explain what is being tested
- Include integration tests for user-facing features

## File Organization

### Project Structure
```
ai-friendly-tools/
├── .github/                 # GitHub configuration and workflows
│   ├── workflows/          # CI/CD workflows
│   └── copilot-instructions.md
├── core/                   # Core library package (workspace member)
│   ├── src/               # Source code
│   ├── pyproject.toml     # Package configuration
│   └── README.md          # Core documentation
├── web/                   # Web tools package (workspace member)
│   ├── src/
│   ├── pyproject.toml
│   └── README.md
├── mcp-manager/           # MCP manager package (workspace member)
│   ├── src/
│   ├── pyproject.toml
│   └── README.md
├── memo/                  # Memory & AI tools package (not in workspace)
│   ├── src/
│   └── pyproject.toml
├── tests/                 # All tests (repository level)
├── docs/                  # Documentation
│   ├── ARCHITECTURE.md
│   ├── DEVELOPMENT.md
│   ├── INSTALLATION.md
│   └── Docker.md
├── Dockerfile            # Docker image definition
├── docker-compose.yml    # Docker services configuration
├── pyproject.toml        # Workspace configuration (Ruff, pytest)
└── uv.lock              # UV lock file
```

### Adding New Code
- **New package modules**: Add to respective package's `src/` directory
- **New tests**: Add to `tests/` directory at repository root
- **Documentation**: Update relevant README.md files and docs/
- **Configuration**: Modify pyproject.toml files as needed

## Development Workflow

### Making Changes
1. Create a feature branch from `main` or `develop`
2. Make changes following code style guidelines
3. Write or update tests for your changes
4. Run linter: `uv run ruff check .`
5. Run formatter: `uv run ruff format .`
6. Run tests: `uv run pytest tests/ -v`
7. Run type checker: `uv run mypy core/src web/src mcp-manager/src --strict` (warnings acceptable)
   - Note: CI also checks memo/src, but it's not in the workspace
8. Commit with descriptive messages
9. Create pull request targeting `main` or `develop`

### CI Pipeline
The CI workflow runs on all pull requests:
- Linting with Ruff (style, errors, imports, naming)
- Format checking with Ruff
- Type checking with MyPy (strict mode, failures are non-blocking)
- Tests with pytest and coverage reporting (workspace packages: core, web, mcp-manager)
- Security scanning with Trivy
- Docker image build (on main branch pushes)

### Dependencies
- Use UV for dependency management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agentic-dev-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
