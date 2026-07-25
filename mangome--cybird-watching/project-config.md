---
trigger: always_on
description: Run linting after every code change:
---

# FastLED Project Rules for Cursor

## Cursor Configuration

### Post-Change Hooks
Run linting after every code change:
```yaml
post_change_hooks:
  - command: "bash lint"
    description: "Run code formatting and linting"
    working_directory: "."
```

## MCP Server Configuration
This project includes a custom MCP server (`mcp_server.py`) that provides tools for:
- Running tests with various options
- Compiling examples for different platforms  
- Code fingerprinting and change detection
- Linting and formatting
- Project information and status
- **Build info analysis for platform-specific defines, compiler flags, and toolchain information**
- **Symbol analysis for binary optimization (all platforms)**
- Stack trace setup for enhanced debugging
- **🌐 FastLED Web Compiler with Playwright (FOREGROUND AGENTS ONLY)**
- **🚨 CRITICAL: `validate_completion` tool for background agents**

To use the MCP server, run: `uv run mcp_server.py`

**BACKGROUND AGENTS:** The MCP server includes a mandatory `validate_completion` tool that MUST be used before indicating task completion. This tool runs `bash test` and ensures all tests pass.

### FastLED Web Compiler (FOREGROUND AGENTS ONLY)

**🌐 FOR INTERACTIVE DEVELOPMENT:** The MCP server includes a `run_fastled_web_compiler` tool that:

**Note:** For direct command-line WASM compilation, see the **WASM Sketch Compilation** section below.

- **Compiles Arduino sketches to WASM** for browser execution
- **Captures console.log output** with playwright automation  
- **Takes screenshots** of running visualizations
- **Monitors FastLED_onFrame calls** to verify proper initialization
- **Provides detailed analysis** of errors and performance

**PREREQUISITES:**
- `pip install fastled` - FastLED web compiler
- `pip install playwright` - Browser automation (included in pyproject.toml)
- Docker (optional, for faster compilation)

**USAGE EXAMPLES:**
```python
# Via MCP Server - Basic usage
use run_fastled_web_compiler tool with:
- example_path: "examples/Audio"
- capture_duration: 30
- headless: false
- save_screenshot: true

# Via MCP Server - Different examples
use run_fastled_web_compiler tool with:
- example_path: "examples/Blink"
- capture_duration: 15
- headless: true

# Via MCP Server - Quick test
use run_fastled_web_compiler tool with:
- example_path: "examples/wasm"
- capture_duration: 10
```

**KEY FEATURES:**
- **Automatic browser installation:** Installs Chromium via playwright
- **Console.log capture:** Records all browser console output with timestamps
- **Error detection:** Identifies compilation failures and runtime errors
- **FastLED monitoring:** Tracks `FastLED_onFrame` calls to verify functionality
- **Screenshot capture:** Saves visualization images with timestamps
- **Docker detection:** Checks for Docker availability for faster builds
- **Background agent protection:** Automatically disabled for CI/background environments

**🚫 BACKGROUND AGENT RESTRICTION:**
This tool is **COMPLETELY DISABLED** for background agents and CI environments. Background agents attempting to use this tool will receive an error message. This is intentional to prevent:
- Hanging processes in automated environments
- Resource conflicts in CI/CD pipelines  
- Interactive browser windows in headless environments

**CONSOLE.LOG CAPTURE PATTERN:**
The tool follows the pattern established in `ci/wasm_test.py` and `ci/ci/scrapers/`:
```javascript
// Example captured console.log patterns:
[14:25:30] log: FastLED_onFrame called: {"frame":1,"leds":100}
[14:25:30] log: Audio worklet initialized
[14:25:31] error: Missing audio_worklet_processor.js
[14:25:31] warning: WebGL context lost
```

**INTEGRATION WITH EXISTING CI:**
- Complements existing `ci/wasm_test.py` functionality
- Uses same playwright patterns as `ci/ci/scrapers/`
- Leverages existing pyproject.toml dependencies
- Compatible with existing Docker-based compilation workflow

## Project Structure
- `src/` - Main FastLED library source code
- `examples/` - Arduino examples demonstrating FastLED usage
- `tests/` - Test files and infrastructure
- `ci/` - Continuous integration scripts
- `docs/` - Documentation

## Key Commands
- `uv run test.py` - Run all tests
- `uv run test.py --cpp` - Run C++ tests only
- `uv run test.py TestName` - Run specific C++ test
  - For example: running test_xypath.cpp would be uv run test.py xypath
- `./lint` - Run code formatting/linting
- `uv run ci/ci-compile.py uno --examples Blink` - Compile examples for specific platform
  - For example (uno): `uv run ci/ci-compile.py uno --examples Blink`
  - For example (esp32dev): `uv run ci/ci-compile.py esp32dev --examples Blink`
  - For example (esp8266): `uv run ci/ci-compile.py esp01 --examples Blink`
  - For example (teensy31): `uv run ci/ci-compile.py teensy31 --examples Blink`
- **WASM Compilation** - Compile Arduino sketches to run in web browsers:
  - `uv run ci/wasm_compile.py examples/Blink -b --open` - Compile Blink to WASM and open browser
  - `uv run ci/wasm_compile.py path/to/your/sketch -b --open` - Compile any sketch to WASM
- **Symbol Analysis** - Analyze binary size and optimization opportunities:
  - `uv run ci/ci/symbol_analysis.py --board uno` - Analyze UNO platform
  - `uv run ci/ci/symbol_analysis.py --board esp32dev` - Analyze ESP32 platform

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mangome/cybird-watching](https://github.com/Mangome/cybird-watching) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
