---
trigger: always_on
description: This document provides coding guidelines and best practices for contributing to Frigate NVR, a complete and local NVR designed for Home Assistant with AI object detection.
---

# Agent Instructions for Frigate NVR

This document provides coding guidelines and best practices for contributing to Frigate NVR, a complete and local NVR designed for Home Assistant with AI object detection.

## Project Overview

Frigate NVR is a realtime object detection system for IP cameras that uses:

- **Backend**: Python 3.13+ with FastAPI, OpenCV, TensorFlow/ONNX
- **Frontend**: React with TypeScript, Vite, TailwindCSS
- **Architecture**: Multiprocessing design with ZMQ and MQTT communication
- **Focus**: Minimal resource usage with maximum performance

## Code Review Guidelines

When reviewing code, do NOT comment on:

- Missing imports - Static analysis tooling catches these
- Code formatting - Ruff (Python) and Prettier (TypeScript/React) handle formatting
- Minor style inconsistencies already enforced by linters

## Python Backend Standards

### Python Requirements

- **Compatibility**: Python 3.13+
- **Language Features**: Use modern Python features:
  - Pattern matching
  - Type hints (comprehensive typing preferred)
  - f-strings (preferred over `%` or `.format()`)
  - Dataclasses
  - Async/await patterns

### Code Quality Standards

- **Formatting**: Ruff (configured in `pyproject.toml`)
- **Linting**: Ruff with rules defined in project config
- **Type Checking**: Use type hints consistently
- **Testing**: unittest framework - use `python3 -u -m unittest` to run tests
- **Language**: American English for all code, comments, and documentation
- **Punctuation**: Do not use em dashes in documentation, comments, or strings; reword with standard punctuation (commas, colons, parentheses, or separate sentences)

### Logging Standards

- **Logger Pattern**: Use module-level logger

  ```python
  import logging

  logger = logging.getLogger(__name__)
  ```

- **Format Guidelines**:
  - No periods at end of log messages
  - No sensitive data (keys, tokens, passwords)
  - Use lazy logging: `logger.debug("Message with %s", variable)`
- **Log Levels**:
  - `debug`: Development and troubleshooting information
  - `info`: Important runtime events (startup, shutdown, state changes)
  - `warning`: Recoverable issues that should be addressed
  - `error`: Errors that affect functionality but don't crash the app
  - `exception`: Use in except blocks to include traceback

### Error Handling

- **Exception Types**: Choose most specific exception available
- **Try/Catch Best Practices**:
  - Only wrap code that can throw exceptions
  - Keep try blocks minimal - process data after the try/except
  - Avoid bare exceptions except in background tasks

  Bad pattern:

  ```python
  try:
      data = await device.get_data()  # Can throw
      # ❌ Don't process data inside try block
      processed = data.get("value", 0) * 100
      result = processed
  except DeviceError:
      logger.error("Failed to get data")
  ```

  Good pattern:

  ```python
  try:
      data = await device.get_data()  # Can throw
  except DeviceError:
      logger.error("Failed to get data")
      return

  # ✅ Process data outside try block
  processed = data.get("value", 0) * 100
  result = processed
  ```

### Async Programming

- **External I/O**: All external I/O operations must be async
- **Best Practices**:
  - Avoid sleeping in loops - use `asyncio.sleep()` not `time.sleep()`
  - Avoid awaiting in loops - use `asyncio.gather()` instead
  - No blocking calls in async functions
  - Use `asyncio.create_task()` for background operations
- **Thread Safety**: Use proper synchronization for shared state

### Documentation Standards

- **Module Docstrings**: Concise descriptions at top of files
  ```python
  """Utilities for motion detection and analysis."""
  ```
- **Function Docstrings**: Required for public functions and methods

  ```python
  async def process_frame(frame: ndarray, config: Config) -> Detection:
      """Process a video frame for object detection.

      Args:
          frame: The video frame as numpy array
          config: Detection configuration

      Returns:
          Detection results with bounding boxes
      """
  ```

- **Comment Style**:
  - Explain the "why" not just the "what"
  - Keep lines under 88 characters when possible
  - Use clear, descriptive comments

### File Organization

- **API Endpoints**: `frigate/api/` - FastAPI route handlers
- **Configuration**: `frigate/config/` - Configuration parsing and validation
- **Detectors**: `frigate/detectors/` - Object detection backends
- **Events**: `frigate/events/` - Event management and storage
- **Utilities**: `frigate/util/` - Shared utility functions

## Frontend (React/TypeScript) Standards

### Internationalization (i18n)

- **CRITICAL**: Never write user-facing strings directly in components
- **Always use react-i18next**: Import and use the `t()` function

  ```tsx
  import { useTranslation } from "react-i18next";

  function MyComponent() {
    const { t } = useTranslation(["views/live"]);
    return <div>{t("camera_not_found")}</div>;
  }
  ```

- **Translation Files**: Add English strings to the appropriate json files in `web/public/locales/en`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blakeblackshear/frigate](https://github.com/blakeblackshear/frigate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
