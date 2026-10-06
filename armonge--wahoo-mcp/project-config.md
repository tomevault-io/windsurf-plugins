---
trigger: always_on
description: This is a Model Context Protocol (MCP) server that provides access to Wahoo Cloud API for retrieving workout data.
---

# Wahoo MCP Server - Development Notes

## Project Overview
This is a Model Context Protocol (MCP) server that provides access to Wahoo Cloud API for retrieving workout data.

## Requirements
- Python 3.13+ (strictly enforced)
- uv package manager (recommended) or pip
- Wahoo API credentials (Client ID and Secret)
- Active Wahoo account with workout data

## Quick Start
```bash
# Install dependencies
uv sync

# Run authentication to get tokens
make auth

# Run the MCP server
uv run python -m src.server

# Run tests
pytest -v

# Run code quality checks
pre-commit run --all-files
```

## Key Commands

### Testing
```bash
# Run basic tests
make test

# Run tests with coverage
make test-cov
```

### Code Quality
```bash
# Run all pre-commit hooks
pre-commit run --all-files

# Run ruff linting
make lint

# Run ruff formatting
make format

# Run all checks (lint + format + test)
make check

# Run complete CI workflow
make ci
```

**IMPORTANT**: Always run `make check` or `ruff check .` after making any code updates to ensure code quality standards are met.

### Development Setup
```bash
# Install dependencies (including dev dependencies)
uv sync

# Activate virtual environment
source .venv/bin/activate

# Install pre-commit hooks
pre-commit install
```

## Architecture

### Main Components
- **WahooAPIClient**: HTTP client for Wahoo Cloud API
- **MCP Server**: Provides 8 tools for workouts, routes, plans, and power zones
- **Authentication**: Uses token file specified by `WAHOO_TOKEN_FILE` environment variable

### API Endpoints
- `GET /v1/workouts` - List workouts with pagination and date filters
- `GET /v1/workouts/{id}` - Get detailed workout information
- `GET /v1/routes` - List routes with optional external_id filter
- `GET /v1/routes/{id}` - Get detailed route information
- `GET /v1/plans` - List plans with optional external_id filter
- `GET /v1/plans/{id}` - Get detailed plan information
- `GET /v1/power_zones` - List power zones for the user
- `GET /v1/power_zones/{id}` - Get detailed power zone information

### MCP Tools
1. **list_workouts**: List workouts with optional filters (page, per_page, start_date, end_date)
2. **get_workout**: Get detailed workout information by ID
3. **list_routes**: List routes with optional external_id filter
4. **get_route**: Get detailed route information by ID
5. **list_plans**: List plans with optional external_id filter
6. **get_plan**: Get detailed plan information by ID
7. **list_power_zones**: List power zones for the user
8. **get_power_zone**: Get detailed power zone information by ID

## Testing Strategy

### Test Structure
- **TestWahooAPIClient**: Unit tests for HTTP client functionality
- **TestMCPTools**: Integration tests for MCP tool handlers

### Mock Strategy
- Uses `MagicMock` for HTTP responses to avoid `raise_for_status()` issues
- Patches `WahooAPIClient` methods for MCP tool testing
- Environment variable mocking for token authentication

## Code Quality Setup

### Pre-commit Hooks
- **trailing-whitespace**: Removes trailing whitespace
- **end-of-file-fixer**: Ensures files end with newline
- **check-yaml**: Validates YAML syntax
- **check-added-large-files**: Prevents large file commits
- **ruff**: Linting with auto-fix
- **ruff-format**: Code formatting

### Development Workflow
1. Make changes
2. Run tests: `make test`
3. Run code quality checks: `make check`
4. Pre-commit hooks run automatically on commit
5. All checks must pass before commit

### CI/CD Pipeline
The project uses GitHub Actions for automated testing and quality checks:

- **Pull Requests**: Runs full test suite, linting, and formatting checks
- **Main Branch**: Additional build step after tests pass
- **Security**: Integrated ruff security checks (flake8-bandit rules)
- **Coverage**: Automatic coverage reporting to Codecov

Key features:
- Uses Makefile commands for consistency between local and CI
- Tests on Python 3.13 only (matching project requirements)
- Automatic dependency updates via Dependabot

## Environment Variables
- `WAHOO_TOKEN_FILE`: Required - path to file for persistent token storage
- `WAHOO_CLIENT_ID`: Required for authentication
- `WAHOO_CLIENT_SECRET`: Required for token refresh (confidential clients)

## OAuth Token Management

### Token Expiration
- Access tokens expire after **2 hours**
- Refresh tokens should be used to obtain new access tokens without re-authentication

### Refresh Token Endpoints

#### PKCE Flow (Used by this app)
```
POST https://api.wahooligan.com/oauth/token
Parameters:
- client_id: Your client ID
- grant_type: refresh_token
- refresh_token: Your refresh token
- code_verifier: The PKCE code verifier from initial auth
```

#### Standard OAuth Flow (For confidential apps)
```
POST https://api.wahooligan.com/oauth/token
Parameters:
- client_id: Your client ID
- client_secret: Your client secret
- grant_type: refresh_token
- refresh_token: Your refresh token
```

### Important Notes
- Once a refreshed access token is used, previous tokens are revoked
- Store the new refresh token returned with each refresh
- The code_verifier must be the same one used during initial authentication

## File Structure
```
src/
├── server.py          # Main MCP server implementation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [armonge/wahoo-mcp](https://github.com/armonge/wahoo-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
