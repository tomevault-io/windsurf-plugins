---
trigger: always_on
description: IDA HCLI is a Python command-line tool for managing IDA Pro installations, licenses, downloads, and cloud integrations. It provides a rich CLI interface using modern Python tooling for reverse engineering workflows.
---

# IDA HCLI

IDA HCLI is a Python command-line tool for managing IDA Pro installations, licenses, downloads, and cloud integrations. It provides a rich CLI interface using modern Python tooling for reverse engineering workflows.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites
- **Python 3.10+** - Required for development
- **uv package manager** - Primary dependency management tool

### Bootstrap, Build, and Test the Repository

```bash
# Install uv package manager (if not available)
pip install uv

# Install dependencies - NEVER CANCEL: Takes 30 seconds. Set timeout to 180+ seconds.
uv sync

# Install additional dependency groups as needed
uv sync --extra test        # Takes additional 10 seconds - NEVER CANCEL. Set timeout to 60+ seconds.
uv sync --extra dev         # Takes additional 45 seconds - NEVER CANCEL. Set timeout to 180+ seconds.
uv sync --extra docs        # Takes additional 30 seconds - NEVER CANCEL. Set timeout to 120+ seconds.
```

### Run Tests
```bash
# Run all tests - NEVER CANCEL: Takes 20 seconds. Set timeout to 120+ seconds.
uv run pytest tests/ -v

# Run specific test categories
uv run pytest tests/integration/ -v     # Integration tests
uv run pytest tests/lib/ -v             # Unit tests

# Note: Some tests fail without IDA installation (expected in dev environment)
# Integration tests require HCLI_INTEGRATION_TESTS=1 environment variable
# Download/license tests require HCLI_API_KEY environment variable
```

### Run the CLI Application
```bash
# Always run dependency installation first
uv sync

# Basic CLI usage
uv run hcli --help                      # Show main help
uv run hcli --version                   # Show version

# Test core functionality (without authentication)
uv run hcli whoami                      # Shows login status
uv run hcli auth --help                 # Auth management commands
uv run hcli download --help             # Download IDA binaries
uv run hcli ida --help                  # IDA installation management
uv run hcli license --help              # License management
uv run hcli extension --help            # Extension management
```

### Development Tools
```bash
# Code formatting and linting - ALWAYS run before committing
uv run ruff format --line-length 120 src/ tests/      # Format code
uv run ruff check src/ tests/                         # Lint code
uv run ruff check --select I src/ tests/              # Sort imports
uv run ruff check --fix src/ tests/                   # Auto-fix issues

# Type checking
uv run mypy --check-untyped-defs --ignore-missing-imports src/ tests/

# Build package - NEVER CANCEL: Takes 1-2 seconds. Set timeout to 60+ seconds.
uv build

# Build documentation - Takes 1 second
uv run mkdocs build
uv run mkdocs serve                     # Serve docs locally at http://127.0.0.1:8000/
```

## Validation

### Always Run Development Validation Steps
After making any changes, ALWAYS run these validation steps:

```bash
# 1. Format and lint code
uv run ruff format --line-length 120 src/ tests/
uv run ruff check --fix src/ tests/
uv run ruff check --select I --fix src/ tests/

# 2. Run type checking  
uv run mypy --check-untyped-defs --ignore-missing-imports src/ tests/

# 3. Run tests
uv run pytest tests/ -v

# 4. Test CLI functionality
uv run hcli --help
uv run hcli whoami
```

### Manual Validation Scenarios
ALWAYS test these user scenarios after making changes:

#### Authentication Flow
```bash
# Test authentication commands (without actual login)
uv run hcli whoami                      # Should show "You are not logged in"
uv run hcli auth list                   # Should work without authentication  
uv run hcli auth --help                 # Should show auth command help
```

#### Download Flow
```bash
# Test download command structure
uv run hcli download --help             # Should show download options
uv run hcli download --mode direct --help  # Should show direct mode help
# Note: Actual downloads require authentication (HCLI_API_KEY)
```

#### Extension Management
```bash
# Test extension commands
uv run hcli extension --help            # Should show extension commands
uv run hcli extension list              # Should list available extensions
```

#### Package Validation
```bash
# Test package builds correctly
uv build                               # Should create dist/ directory with wheel and sdist
ls -la dist/                           # Should show ida_hcli-*.whl and ida_hcli-*.tar.gz
```

### CI/CD Validation
The CI pipeline (.github/workflows/) will fail unless these pass:

```bash
# Style checks (required for CI)
uv run ruff format --check --line-length 120 src/ tests/
uv run ruff check src/ tests/
uv run ruff check --select I src/ tests/
uv run mypy --check-untyped-defs --ignore-missing-imports src/ tests/

# Test checks (required for CI)  
uv run pytest tests/ -v
```

## Common Tasks

### Adding New CLI Commands
1. Create command module in appropriate group under `src/hcli/commands/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HexRaysSA/ida-hcli](https://github.com/HexRaysSA/ida-hcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
