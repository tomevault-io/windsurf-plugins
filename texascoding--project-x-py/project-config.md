---
trigger: always_on
description: This file provides guidance to Cursor AI when working with code in this repository.
---

# ProjectX Python SDK - Cursor AI Rules

This file provides guidance to Cursor AI when working with code in this repository.

## 🚨 CRITICAL: Focused Development Rules

**MANDATORY**: This project follows strict development rules located in `.cursor/rules/`:

- **[TDD Core Rules](.cursor/rules/tdd_core.md)** - Test-Driven Development enforcement
- **[Async Testing Rules](.cursor/rules/async_testing.md)** - Async-first testing patterns
- **[Code Quality Rules](.cursor/rules/code_quality.md)** - Quality standards and validation
- **[Development Workflow](.cursor/rules/development_workflow.md)** - Process enforcement

**READ THESE RULES FIRST** before making any code changes. All rules are MANDATORY and must be followed strictly.

## Project Status: v3.2.0 - Enhanced Type Safety Release

**IMPORTANT**: This project uses a fully asynchronous architecture. All APIs are async-only, optimized for high-performance futures trading.

## Development Phase Guidelines

**IMPORTANT**: This project has reached stable production status. When making changes:

1. **Maintain Backward Compatibility**: Keep existing APIs functional with deprecation warnings
2. **Deprecation Policy**: Mark deprecated features with warnings, remove after 2 minor versions
3. **Semantic Versioning**: Follow semver strictly (MAJOR.MINOR.PATCH)
4. **Migration Paths**: Provide clear migration guides for breaking changes
5. **Modern Patterns**: Use the latest Python patterns while maintaining compatibility
6. **Gradual Refactoring**: Improve code quality without breaking existing interfaces
7. **Async-First**: All new code must use async/await patterns

## Non-Negotiable TDD Requirements

**ALWAYS follow RED-GREEN-REFACTOR cycle:**
1. 🔴 RED: Write failing test defining expected behavior
2. 🟢 GREEN: Write minimal code to make test pass
3. 🔄 REFACTOR: Improve code while keeping tests green

**Tests are the specification. Code must conform to tests, not vice versa.**

Example approach:
- ✅ DO: Keep old method signatures with deprecation warnings
- ✅ DO: Provide new improved APIs alongside old ones
- ✅ DO: Add compatibility shims when necessary
- ✅ DO: Document migration paths clearly
- ❌ DON'T: Break existing APIs without major version bump
- ❌ DON'T: Remove deprecated features without proper notice period

### Deprecation Process
1. Use the standardized `@deprecated` decorator from `project_x_py.utils.deprecation`
2. Provide clear reason, version info, and replacement path
3. Keep deprecated feature for at least 2 minor versions
4. Remove only in major version releases (4.0.0, 5.0.0, etc.)

Example:
```python
from project_x_py.utils.deprecation import deprecated, deprecated_class

# For functions/methods
@deprecated(
    reason="Method renamed for clarity",
    version="3.1.14",  # When deprecated
    removal_version="4.0.0",  # When it will be removed
    replacement="new_method()"  # What to use instead
)
def old_method(self):
    return self.new_method()

# For classes
@deprecated_class(
    reason="Integrated into TradingSuite",
    version="3.1.14",
    removal_version="4.0.0",
    replacement="TradingSuite"
)
class OldManager:
    pass
```

The standardized deprecation utilities provide:
- Consistent warning messages across the SDK
- Automatic docstring updates with deprecation info
- IDE support through the `deprecated` package
- Metadata tracking for deprecation management
- Support for functions, methods, classes, and parameters

## Development Documentation

### Important: Keep Project Clean - Use External Documentation

**DO NOT create project files for**:
- Personal development notes
- Temporary planning documents
- Testing logs and results
- Work-in-progress documentation
- Meeting notes or discussions

**Instead, use**:
- External documentation tools (Obsidian, Notion, etc.)
- GitHub Issues for bug tracking
- GitHub Discussions for architecture decisions
- Pull Request descriptions for implementation details

This keeps the project repository clean and focused on production code.

## Development Commands

### Package Management (UV)
```bash
uv add [package]              # Add a dependency
uv add --dev [package]        # Add a development dependency
uv sync                       # Install/sync dependencies
uv run [command]              # Run command in virtual environment
```

### Testing
```bash
uv run pytest                # Run all tests
uv run pytest tests/test_client.py  # Run specific test file
uv run pytest -m "not slow"  # Run tests excluding slow ones
uv run pytest --cov=project_x_py --cov-report=html  # Generate coverage report
uv run pytest -k "async"     # Run only async tests
```

### Async Testing Patterns
```python
# Test async methods with pytest-asyncio
import pytest

@pytest.mark.asyncio
async def test_async_method():
    async with ProjectX.from_env() as client:
        await client.authenticate()
        result = await client.get_bars("MNQ", days=1)
        assert result is not None
```

### Code Quality
```bash
uv run ruff check .          # Lint code
uv run ruff check . --fix    # Auto-fix linting issues
uv run ruff format .         # Format code
uv run mypy src/             # Type checking
```

### Building and Distribution
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TexasCoding/project-x-py](https://github.com/TexasCoding/project-x-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
