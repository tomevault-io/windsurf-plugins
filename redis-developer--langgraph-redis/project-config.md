---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL: Integration Tests First, Always

**When fixing bugs or adding features, ALWAYS write and run integration tests with real Redis BEFORE declaring the fix
complete.** Unit tests with mocks are NOT sufficient — mocks hide real bugs. If a fix only passes mocked unit tests, it
is NOT verified. Write the integration test first, see it fail, fix the code, then see it pass.

## How to Write Tests for Coverage

When improving test coverage, follow these principles:

1. **Focus on Integration Tests**: Write tests that use real Redis instances and test actual usage scenarios. Unit tests
   are secondary to integration tests.

2. **Test What Code SHOULD Do**: Don't write tests that mirror what the code currently does. Test against the expected
   behavior and requirements.

3. **Use Meaningful Test Names**: Test names should describe the behavior being tested, not generic names like "
   test_function_x".

4. **Research Before Writing**: Find and understand existing tests for the feature/area before adding new tests.

5. **Test Error Paths and Edge Cases**: Focus on uncovered error handling, boundary conditions, and edge cases.

6. **Run Tests Incrementally**: Run `make test-all` after every 5 tests to ensure no regressions.

7. **Avoid "Ugly Mirror" Testing**: Don't create tests that simply verify the current implementation. Test the contract
   and expected behavior.

Example of a good integration test for error handling:

```python
def test_malformed_base64_blob_handling(redis_url: str) -> None:
    """Test handling of malformed base64 data in blob decoding."""
    with _saver(redis_url) as saver:
        # Set up real scenario
        # Test error condition
        # Verify graceful handling
```

## CRITICAL: Always Use TestContainers for Redis

**NEVER use Docker directly or manually start Redis containers!** All tests, benchmarks, and profiling scripts MUST use
TestContainers. The library handles container lifecycle automatically.

```python
from testcontainers.redis import RedisContainer

# Use redis:8 (has all required modules) or redis/redis-stack-server:latest
redis_container = RedisContainer("redis:8")
redis_container.start()
try:
    redis_url = f"redis://{redis_container.get_container_host_ip()}:{redis_container.get_exposed_port(6379)}"
    # Use redis_url...
finally:
    redis_container.stop()
```

## Development Commands

### Setup and Dependencies

```bash
poetry install --all-extras  # Install all dependencies with poetry (from README)
```

### Testing

```bash
make test-all               # PREFERRED: Run all tests including API tests when evaluating changes
make test                   # Run tests with verbose output
make test-coverage          # Run tests with coverage
make coverage-report        # Show coverage report in terminal
make coverage-html          # Generate HTML coverage report
pytest tests/test_specific.py  # Run specific test file
pytest tests/test_specific.py::test_function  # Run specific test
pytest --run-api-tests      # Include API integration tests
```

**Important**: Always use `make test-all` when evaluating changes to ensure all tests pass, including API integration
tests.

Note: Tests automatically use TestContainers for Redis - do not manually start Redis containers.

### Code Quality

```bash
make format          # Format code with black and isort
make lint            # Run formatting, type checking, and other linters
make check-types     # Run mypy type checking
make check           # Run both linting and tests
make find-dead-code  # Find unused code with vulture
poetry run check-format      # Check formatting without modifying
poetry run check-sort-imports # Check import sorting
poetry run check-lint        # Run all linting checks
```

### Development

```bash
make clean           # Remove cache and build artifacts
```

## Code Style Guidelines

- Use Black for formatting with target versions py39-py313
- Sort imports with isort (black profile)
- Strict typing required (disallow_untyped_defs=True)
- Follow PEP 8 naming conventions (snake_case for functions/variables)
- Type annotations required for all function parameters and return values
- Explicit error handling with descriptive error messages
- Test all functionality with both sync and async variants
- Maintain test coverage with pytest
- Use contextlib for resource management
- Document public APIs with docstrings

## Architecture Overview

### Core Components

**Checkpoint Savers** (`langgraph/checkpoint/redis/`):

- `base.py`: `BaseRedisSaver` - Abstract base class with shared Redis operations, schemas, and TTL management
- `__init__.py`: `RedisSaver` - Standard sync implementation
- `aio.py`: `AsyncRedisSaver` - Async implementation
- `shallow.py` / `ashallow.py`: Shallow variants that store only latest checkpoint
- `key_registry.py`: Checkpoint key registry using sorted sets for efficient write tracking
- `scan_utils.py`: Utilities for efficient key scanning and pattern matching

**Stores** (`langgraph/store/redis/`):

- `base.py`: `BaseRedisStore` - Abstract base with Redis operations, vector search, and TTL support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redis-developer/langgraph-redis](https://github.com/redis-developer/langgraph-redis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
