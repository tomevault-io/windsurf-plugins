---
trigger: always_on
description: This document provides essential guidelines for AI agents working on the alpha-trading-bot-okx repository.
---

# AGENTS.md

This document provides essential guidelines for AI agents working on the alpha-trading-bot-okx repository.

## Project Overview

Python 3.8+ cryptocurrency trading bot for OKX exchange with modular architecture, AI-driven signals, and comprehensive risk management.

## Build, Lint, Test Commands

```bash
# Install development dependencies
pip install -e ".[dev]"

# Run all tests
pytest

# Run single test file
pytest tests/unit/test_core.py

# Run specific test function
pytest tests/unit/test_core.py::test_function_name

# Run tests with coverage
pytest --cov=alpha_trading_bot --cov-report=term-missing

# Code formatting
black alpha_trading_bot/

# Type checking
mypy alpha_trading_bot/

# Linting
flake8 alpha_trading_bot/ --max-line-length=88 --extend-ignore=E203,W503

# Import sorting
isort alpha_trading_bot/ --profile black

# Run all pre-commit hooks
pre-commit run --all-files
```

## Code Style Guidelines

### Imports
- Use `isort` with `--profile black` for sorting
- Group imports: standard library → third-party → local
- Use absolute imports for project modules: `from alpha_trading_bot.core import BaseComponent`
- Avoid relative imports beyond single level: `from .base import BaseComponent`

### Formatting
- **Black formatter** with `line-length = 88`
- **No trailing whitespace** (enforced by pre-commit)
- **File must end with newline** (enforced by pre-commit)
- **Use 4 spaces for indentation** (Python standard)

### Type System
- **Strict type checking** required (mypy config enforces this)
- Always add type hints to function signatures: `async def calculate(self, price: float) -> Optional[float]`
- Use typing module: `from typing import Dict, Any, Optional, List, Tuple, Callable`
- Never suppress type errors with `# type: ignore`
- Import from dataclasses: `from dataclasses import dataclass`

### Naming Conventions
- **Classes**: PascalCase (`TradingBot`, `ConfigManager`)
- **Functions/Methods**: snake_case (`calculate_atr`, `get_market_data`)
- **Variables**: snake_case (`market_data`, `api_key`)
- **Constants**: UPPER_SNAKE_CASE (`CONFIDENCE_THRESHOLD_LOW`, `MAX_RETRIES`)
- **Private members**: single underscore prefix (`_cache`, `_initialize`)
- **Protected members**: single underscore prefix
- **Private classes**: single underscore prefix (rare, avoid if possible)

### Error Handling
- Use custom exceptions from `alpha_trading_bot.core.exceptions`:
  - `TradingBotException` (base exception)
  - `ConfigurationError`
  - `ExchangeError`
  - `StrategyError`
  - `RiskControlError`
  - `AIProviderError`
  - `NetworkError`
  - `RateLimitError`
- Always include descriptive error messages with context
- Use `try/except/finally` for resource cleanup
- Never use bare `except:` (always specify exception type)
- Log errors before raising: `logger.error(f"Failed to initialize: {e}")`

### Logging
- Use standard `logging` module
- Get logger at module level: `logger = logging.getLogger(__name__)`
- Use appropriate log levels:
  - `DEBUG`: Detailed debugging information
  - `INFO`: General information about execution flow
  - `WARNING`: Something unexpected but recoverable
  - `ERROR`: Error that prevented an operation
- Include relevant context in log messages
- For enhanced logging, use `LoggerMixin` from `alpha_trading_bot.utils.logging`

### Architecture Patterns
- **Base Classes**: Inherit from `BaseComponent` and `BaseConfig` (from `alpha_trading_bot.core.base`)
- **Data Models**: Use `@dataclass` for configuration and data structures
- **Async Operations**: Use `asyncio` with proper `await` syntax
- **Retry Logic**: Use `@retry_on_network_error` decorator for network operations (defined in `exchange/client.py`)
- **Configuration**: Use `ConfigManager` from `alpha_trading_bot.config` for loading configs
- **Caching**: Use `cache_manager` or `DynamicCacheManager` for AI signal caching (15-minute default)

### Module Structure
- **core/**: Base classes, bot logic, exceptions, health checks, monitoring
- **config/**: Configuration management (ConfigManager, models)
- **exchange/**: Trading engine, client, order/position/risk management
- **strategies/**: Trading strategies, consolidation detection, low-price strategy
- **ai/**: AI providers (kimi, deepseek, qwen, openai), fusion, signal generation
- **utils/**: Technical indicators, logging, cache, crash detection, price calculator
- **data/**: Database management, models, data access layer
- **api/**: REST API endpoints, client wrappers
- **cli/**: Command-line interface

### Documentation
- **Docstrings**: Use Google-style docstrings
  ```python
  def calculate_atr(high: List[float], low: List[float], close: List[float]) -> List[float]:
      """
      Calculate Average True Range (ATR).

      Args:
          high: List of high prices
          low: List of low prices
          close: List of close prices

      Returns:
          List of ATR values

      Raises:
          ValueError: If input lists have different lengths
      """
  ```
- **Comments**: Use Chinese comments for business logic explanations (project standard)
- **TODO/FIXME**: Add these sparingly with clear descriptions

### Testing Guidelines
- Use `pytest` for all tests
- Use `pytest-asyncio` for async tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cl173339546/alpha-trading-bot-okx](https://github.com/cl173339546/alpha-trading-bot-okx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
