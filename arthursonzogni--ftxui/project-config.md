---
trigger: always_on
description: FTXUI is a cross-platform C++ library for terminal-based user interfaces with a functional programming approach, inspired by React.
---

# FTXUI - Functional Terminal (X) User Interface

FTXUI is a cross-platform C++ library for terminal-based user interfaces with a functional programming approach, inspired by React.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the information here.**

## Working Effectively

### Build System Setup and Commands
- Bootstrap and build the repository:
  ```bash
  # Basic build (library only) - fast
  cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
  cmake --build build --parallel $(nproc)
  # Build time: ~30 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
  
  # Build with examples  
  cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DFTXUI_BUILD_EXAMPLES=ON
  cmake --build build --parallel $(nproc)
  # Build time: ~70 seconds. NEVER CANCEL. Set timeout to 180+ seconds.
  
  # Build with examples and tests
  cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DFTXUI_BUILD_EXAMPLES=ON -DFTXUI_BUILD_TESTS=ON
  cmake --build build --parallel $(nproc)
  # Build time: ~113 seconds (includes GoogleTest download). NEVER CANCEL. Set timeout to 300+ seconds.
  ```

- Alternative build with Ninja (faster):
  ```bash
  cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DFTXUI_BUILD_EXAMPLES=ON
  ninja -C build
  # Build time: ~62 seconds. NEVER CANCEL. Set timeout to 180+ seconds.
  ```

- Run unit tests:
  ```bash
  # Configure with tests enabled first, then:
  cd build && ctest --output-on-failure
  # Test time: ~4 seconds (302 tests). NEVER CANCEL. Set timeout to 60+ seconds.
  ```

### Bazel Support
- FTXUI also supports Bazel build system
- **WARNING**: Bazel may fail due to network connectivity issues in sandboxed environments
- If Bazel is available:
  ```bash
  bazel build //...  # Build everything
  bazel test //...   # Run tests
  ```

## Validation

### Manual Testing After Changes
- **ALWAYS manually validate changes by building and running examples after making code modifications**
- Run example applications to verify functionality:
  ```bash
  # Build an example first
  cmake --build build --target ftxui_example_border
  
  # Run examples (they are interactive, use timeout to terminate)
  timeout 2s build/examples/dom/ftxui_example_border
  timeout 2s build/examples/dom/ftxui_example_color_gallery
  timeout 2s build/examples/component/ftxui_example_button
  ```
- Examples should produce visual terminal output with borders, colors, and UI components
- **CRITICAL**: Always run at least one DOM example and one Component example to verify both modules work

### Code Quality and Formatting
- Always run formatting before committing:
  ```bash
  ./tools/format.sh
  # Format time: ~7 seconds. NEVER CANCEL. Set timeout to 60+ seconds.
  ```
- The format script adds license headers and runs clang-format on all source files
- **Required**: Run formatting or the CI (.github/workflows/build.yaml) will fail

### Build Validation Requirements
- ALWAYS build with both `-DFTXUI_BUILD_EXAMPLES=ON` and `-DFTXUI_BUILD_TESTS=ON` when making changes
- Run the complete test suite with `ctest --output-on-failure`
- All 302 tests must pass
- **Scenario Testing**: Run at least these validation scenarios:
  1. Build library only (basic validation)
  2. Build with examples and run 2-3 different examples
  3. Build with tests and run complete test suite
  4. Run formatting tool to ensure code style compliance

## Project Structure

### Key Directories
```
/home/runner/work/FTXUI/FTXUI/
├── include/ftxui/           # Public header files
│   ├── component/           # Interactive component headers
│   ├── dom/                 # DOM element headers  
│   ├── screen/              # Screen and rendering headers
│   └── util/                # Utility headers
├── src/ftxui/               # Implementation files
│   ├── component/           # Interactive components (buttons, menus, etc.)
│   ├── dom/                 # DOM elements (layout, styling, text)
│   ├── screen/              # Screen rendering and terminal handling
│   └── util/                # Utilities
├── examples/                # Example applications
│   ├── component/           # Interactive component examples
│   └── dom/                 # DOM element examples
├── cmake/                   # CMake configuration files
├── tools/                   # Development tools (formatting, etc.)
└── .github/workflows/       # CI/CD configuration
```

### Core Library Modules
FTXUI is organized into three main modules that depend on each other:
```
┌component──┐  (Interactive UI components)
│┌dom──────┐│  (Layout and styling elements)  
││┌screen─┐││  (Terminal rendering and input)
└┴┴───────┴┴┘
```

1. **screen**: Low-level terminal handling, colors, pixels, input
2. **dom**: Layout elements (hbox, vbox, text, borders, etc.)
3. **component**: Interactive components (buttons, menus, input fields)

### CMake Build Options
| Option                            | Description                      | Default |
|-----------------------------------|----------------------------------|---------|
| FTXUI_BUILD_EXAMPLES              | Build example applications       | OFF     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArthurSonzogni/FTXUI](https://github.com/ArthurSonzogni/FTXUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
