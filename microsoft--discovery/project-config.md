---
trigger: always_on
description: This is a Python-based API polling application designed for portability across different environments (local development, Docker containers, and cloud platforms like Azure Discovery).
---

# Copilot Instructions for Python API Polling Project

## Project Context
This is a Python-based API polling application designed for portability across different environments (local development, Docker containers, and cloud platforms like Azure Discovery).

## Code Style Guidelines

### Python Standards
- Use Python 3.9+ features and type hints for all function signatures
- Follow PEP 8 naming conventions (snake_case for functions/variables, UPPER_CASE for constants)
- Use `pathlib.Path` instead of `os.path` for file operations
- Prefer f-strings for string formatting
- Add docstrings to all functions using Google style format

### Configuration Management
- Use environment variables for all configuration with sensible defaults
- Support both `.env` files (via python-dotenv) and direct environment variables
- Implement configuration precedence: CLI args > env vars > config files > defaults
- Use `pydantic` Settings classes for configuration validation when possible

### API Interaction Patterns
```python
# Always use exponential backoff for retries
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
async def call_api(endpoint: str, **kwargs) -> dict:
    """Make API call with automatic retry logic."""
    pass
```

### Polling Implementation
- Use async/await patterns with `asyncio` for efficient polling
- Implement configurable poll intervals with jitter to avoid thundering herd
- Always include timeout mechanisms
- Log poll attempts and results at appropriate levels (DEBUG for success, WARNING for retries, ERROR for failures)

### Error Handling
- Use custom exception classes for different error scenarios
- Always catch and log exceptions at the appropriate level
- Implement graceful shutdown handling (SIGTERM, SIGINT)
- Return meaningful exit codes (0=success, 1=general error, 2=config error, 3=connection error)

### Logging
```python
import logging
import sys

# Configure structured logging
logging.basicConfig(
    level=logging.INFO,
    format='[%(asctime)s] [%(name)s] [%(levelname)s] %(message)s',
    handlers=[logging.StreamHandler(sys.stdout)]
)
```

### Dependency Management
- Keep dependencies minimal for portability
- Use `pyproject.toml` with Poetry for dependency management
- Pin major versions but allow minor updates (e.g., `requests>=2.28,<3.0`)

### Docker Considerations
- Run as non-root user (UID 1000)
- Use multi-stage builds to minimize image size
- Copy only necessary files (use .dockerignore)
- Set PYTHONUNBUFFERED=1 for proper log streaming

### Security Practices
- Never hardcode secrets or credentials
- Support multiple authentication methods (managed identity, service principal, API keys)
- Use Azure Key Vault or similar for secret management when available
- Validate and sanitize all external inputs
- Use HTTPS for all API calls

### Testing
- Write unit tests for all business logic using `pytest`
- Always write unit tests
- Mock external API calls in tests
- Aim for >80% code coverage
- Include integration tests that can run against test endpoints

### Linting & Type Checking
- Always run linters after substantive edits.
- Use Ruff for linting and formatting fixes; prefer `ruff --fix` on staged/changed files.
- Use MyPy for type checking with the repo's `pyproject.toml` settings.
- Quality gate before completing a task:
    - Lint: Ruff passes (or fixes applied)
    - Typecheck: MyPy passes on `src/` and relevant tests
    - Tests: Pytest passes for affected modules
    - No new warnings/errors introduced by changes
-
Example quick checks (conceptually run by the assistant):
    - ruff check --fix src tests
    - mypy src tests

### Project Structure
```
project/
├── src/
│   ├── __init__.py
│   ├── config.py       # Configuration management
│   ├── client.py       # API client implementation
│   ├── poller.py       # Polling logic
│   └── utils.py        # Helper functions
├── tests/
│   ├── test_client.py
│   ├── test_poller.py
│   └── fixtures/
├── scripts/
│   ├── poll.sh         # Bash wrapper for polling
│   └── run-local.sh    # Local development runner
├── .env.example        # Example environment variables
├── requirements.txt    # Production dependencies
├── Dockerfile          # Container definition
└── README.md          # Usage documentation
```

### CLI Design
- Use `typer` for CLI parsing
- Support both flags and environment variables
- Provide --help with clear examples
- Include --debug flag for verbose output
- Support --dry-run for testing configuration

### Portability Checklist
- [ ] Works on Linux, macOS, and Windows (WSL)
- [ ] Runs in Docker containers
- [ ] Supports both local and cloud environments
- [ ] Handles different authentication methods
- [ ] Configurable via environment variables
- [ ] Graceful degradation when optional services unavailable
- [ ] Clear error messages for missing dependencies

## Example Implementation Pattern
```python
async def poll_api(
    endpoint: str,
    poll_interval: int = 5,
    max_attempts: int = None,
    timeout: int = 300
) -> dict:
    """
    Poll an API endpoint until completion or timeout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/discovery](https://github.com/microsoft/discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
