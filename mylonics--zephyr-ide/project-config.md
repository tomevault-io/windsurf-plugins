---
trigger: always_on
description: **ALWAYS follow these instructions first and only fallback to search or bash commands when encountering unexpected information that does not match the info here.**
---

# Zephyr IDE VS Code Extension

**ALWAYS follow these instructions first and only fallback to search or bash commands when encountering unexpected information that does not match the info here.**

Zephyr IDE is a VS Code extension that provides comprehensive tools for Zephyr RTOS development. It includes workspace setup, SDK installation, project creation, building, debugging, and testing capabilities.

## Working Effectively

### Bootstrap and Dependencies

**CRITICAL: All build and test commands take significant time. NEVER CANCEL operations.**

```bash
# Install Node.js dependencies (40 seconds)
npm install

# Install Python west tool for Zephyr development
pip3 install west

# Verify required tools are available
which python3 && python3 --version
which cmake && cmake --version
which west && west --version
which ninja && ninja --version
```

### Build Commands

**Set timeouts to 10+ minutes for all build commands. NEVER CANCEL.**

```bash
# Compile TypeScript (3 seconds)
npm run test-compile

# Compile for development (3 seconds)
npm run compile

# Bundle extension with esbuild (1 second)
npm run esbuild

# Production build with minification (1 second)
npm run vscode:prepublish

# Run ESLint (2 seconds)
npm run lint

# Full pretest pipeline: compile + esbuild + lint (6 seconds)
npm run pretest
```

### Testing Commands

**CRITICAL: Integration tests take 10-15 minutes each. Set timeouts to 20+ minutes. NEVER CANCEL.**

```bash
# Run all tests (requires VS Code download, network-dependent)
# TIMEOUT: 20+ minutes
npm test

# Run specific integration test suites (requires Zephyr SDK)
# TIMEOUT: 15+ minutes each - NEVER CANCEL
xvfb-run -a node scripts/run-integration-tests.js standard
xvfb-run -a node scripts/run-integration-tests.js west-git
xvfb-run -a node scripts/run-integration-tests.js zephyr-ide-git
xvfb-run -a node scripts/run-integration-tests.js local-west
xvfb-run -a node scripts/run-integration-tests.js external-zephyr

# Skip actual Zephyr builds in tests (faster testing)
SKIP_BUILD_TESTS=true npm test
```

### VS Code Extension Development

```bash
# Open extension for development
# 1. Open VS Code in the repository directory
# 2. Press F5 or Run → Start Debugging
# 3. Select "Run Extension" configuration
# This launches a new VS Code window with the extension loaded for testing
```

## Validation Scenarios

**ALWAYS test these scenarios after making changes to ensure functionality:**

### Manual Extension Validation

1. **Extension Loading**: Press F5 to launch extension host, verify Zephyr IDE appears in activity bar
2. **Workspace Setup**: Test workspace initialization commands work
3. **Project Creation**: Verify project creation from templates functions
4. **Build Configuration**: Test build configuration setup
5. **Command Palette**: Verify all "Zephyr IDE:" commands are available

### Integration Test Coverage

The integration tests validate complete workflows:

- **Standard** (`workspace-standard.test.ts`): Dependencies → west setup → SDK install → project creation → build (15 min)
- **West Git** (`workspace-west-git.test.ts`): West manifest git clone → SDK → project → custom board build (15 min)
- **Zephyr IDE Git** (`workspace-zephyr-ide-git.test.ts`): Zephyr IDE sample project git workflow (15 min)
- **Local West** (`workspace-local-west.test.ts`): Git clone with detected west.yml → local west workspace → build (15 min)
- **External Zephyr** (`workspace-external-zephyr.test.ts`): Git clone without west → external Zephyr installation → build (15 min)
- **Combined** (`combined-installation.test.ts`): Host tool install + standard workspace in single process (25 min)

## Repository Structure

### Core Source Files

```
src/
├── extension.ts         - Main extension entry point, registers commands and webview panels
├── defines.ts          - Zephyr toolchain targets, HALs, and UI dropdown definitions
├── panels/             - Webview panels for different aspects of the UI
│   ├── active_project_view/    - Shows current project build status and controls
│   ├── extension_setup_view/   - Initial workspace setup and SDK installation
│   ├── host_tool_install_view/ - Host tool installation wizard
│   ├── project_config_view/    - Project-specific configuration (boards, runners, etc.)
│   ├── project_tree_view/      - File explorer for Zephyr projects
│   ├── setup_panel/           - Workspace configuration and tool validation
│   ├── west_workspace_view/   - West workspace management panel
│   └── view.css               - Shared CSS styles for all webview panels
├── project_utilities/  - Project-level operations and configuration
│   ├── build_selector.ts      - Board and target selection logic
│   ├── config_selector.ts     - Kconfig and devicetree configuration
│   ├── project.ts             - Core project management and build operations
│   ├── runner_selector.ts     - Flash/debug runner configuration
│   └── twister_selector.ts    - Test runner configuration
├── setup_utilities/    - Workspace setup and dependency management
│   ├── dts_interface.ts       - Devicetree parsing and intellisense
│   ├── host_tools.ts          - Host tool detection and validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mylonics/zephyr-ide](https://github.com/mylonics/zephyr-ide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
