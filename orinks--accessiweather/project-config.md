---
trigger: always_on
description: AccessiWeather is a cross-platform, accessible desktop weather application built with BeeWare/Toga (v0.9+) framework and Briefcase for packaging. The app targets Python 3.10+ (3.12 recommended) and prioritizes screen reader accessibility, multi-source weather data, and a simple architecture.
---

# AccessiWeather AI Coding Agent Instructions

## Project Overview
AccessiWeather is a cross-platform, accessible desktop weather application built with BeeWare/Toga (v0.9+) framework and Briefcase for packaging. The app targets Python 3.10+ (3.12 recommended) and prioritizes screen reader accessibility, multi-source weather data, and a simple architecture.

## Core Architecture

### Weather Data Sources (Multi-Source Strategy)
- **Primary**: NWS API (weather.gov) for US locations - most accurate, no API key required
- **Fallback**: Open-Meteo for NWS failures or international locations - free, no API key
- **Enrichment**: Visual Crossing for enhanced alert data - requires API key, optional
- **Data Flow**: `WeatherClient` coordinates sources, `WeatherDataCache` caches responses (5-minute default)
- **Alert System**: `AlertManager` + `AlertNotificationSystem` handle notifications with rate limiting and severity filtering

### Application Structure
- **Entry Point**: `src/accessiweather/app.py` - Main `AccessiWeatherApp(toga.App)` class
- **Config Management**: `ConfigManager` (JSON-based, uses `toga.App.paths.config`)
  - Portable mode supported: checks for `portable.txt` flag to use local config directory
  - Settings: `AppSettings` model with validation in `config/settings.py`
  - Locations: `LocationOperations` in `config/locations.py`
- **UI Builder**: `ui_builder.py` creates main window and menu system
- **Display Layer**: `WeatherPresenter` formats weather data for screen readers
- **Background Tasks**: Periodic updates via `asyncio.create_task` in `background_tasks.py`

### API Structure
- `api/nws/`: National Weather Service API wrappers (alerts, forecasts, gridpoints)
- `api/openmeteo_wrapper.py`: Open-Meteo integration
- `api/visualcrossing/`: Visual Crossing API (alerts, historical data)
- `api/base_wrapper.py`: Shared HTTP client patterns with retries

## Toga Framework Specifics

### Critical Patterns
1. **OptionContainer API**: Use `option_container.content.append(title, widget)` with TWO separate arguments, NOT `.add(title, widget)` or tuple format
2. **Accessibility**: ALL UI elements MUST have `aria_label` and `aria_description` attributes
3. **Testing**: Use Toga dummy backend (`toga_dummy` package) for unit tests
   - Set `TOGA_BACKEND=toga_dummy` environment variable
   - Fallback to `toga_winforms` on Windows if dummy unavailable
4. **Async Operations**: Toga uses `asyncio` - all long-running operations must be async
5. **Main Thread**: UI updates must happen on main thread, use `app.add_background_task()` for background work

### Common Gotchas
- OptionContainer: Use `.content.append(title, widget)` with two arguments, NOT tuple format `(title, widget)`
- Toga Selection widget (dropdown) uses `.items` list and `.value` property
- **Error Dialogs**: Use built-in `await app.main_window.error_dialog("Title", "Message")` instead of custom dialog classes
- **Info Dialogs**: Use `await app.main_window.info_dialog("Title", "Message")` for informational messages
- **Question Dialogs**: Use `await app.main_window.question_dialog("Title", "Question")` for yes/no prompts
- Modal dialogs: Create with `toga.Window`, show with `.show()`, close with `.close()`
- Focus management: Call `.focus()` on widgets for accessibility (may fail silently on some platforms)

## Development Workflow

### Local Development
```bash
# Run app in development mode (hot reload)
briefcase dev

# Run tests (uses pytest with Toga dummy backend)
pytest -v -p pytest_asyncio

# Auto-fix linting issues and format code (run BEFORE committing)
ruff check --fix .
ruff format .

# Or run both in one line (recommended workflow)
ruff check --fix . && ruff format .

# Type checking (optional, can be noisy)
pyright
```

### Pre-Commit Hooks
- Auto-formats code with `ruff format` (line length 100)
- Auto-fixes linting issues with `ruff check --fix`
- Runs last-failed unit tests (`pytest --lf --ff -m "unit"`)
- Type checks with pyright (excludes tests/)
- **Important**: Pre-commit will auto-stage formatted files

### Build & Package
```bash
# Create platform-specific app bundle
briefcase create

# Build installer (MSI on Windows, DMG on macOS, AppImage on Linux)
briefcase build

# Package for distribution
briefcase package
```

### Testing Conventions
- **Unit tests**: Fast, isolated tests in `tests/test_*.py`
- **Integration tests**: Test real API calls (marked with `@pytest.mark.integration`)
- **Fixtures**: Reusable test fixtures in `tests/toga_test_helpers.py`
  - `DummyConfigManager`: Mock config manager for testing
  - `WeatherDataFactory`: Creates mock weather data
  # AccessiWeather — AI coding agent quick guide

  This file highlights the repository-specific patterns, workflows and commands an AI coding agent should follow to be immediately productive.

  **Architecture & key locations**
  - Entry: `src/accessiweather/app.py` (main `AccessiWeatherApp`)
  - API wrappers: `api/` (see `api/base_wrapper.py` for shared HTTP client patterns)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orinks/AccessiWeather](https://github.com/Orinks/AccessiWeather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
