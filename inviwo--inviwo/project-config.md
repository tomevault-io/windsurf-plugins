---
trigger: always_on
description: **Inviwo** (Interactive Visualization Workshop) is an open-source visualization framework written in **C++23** with **Python** bindings. It enables scientists and developers to build data processing and visualization pipelines through a visual programming (node-graph) interface.
---

# Inviwo Coding Agent Instructions

## Project Overview

**Inviwo** (Interactive Visualization Workshop) is an open-source visualization framework written in **C++23** with **Python** bindings. It enables scientists and developers to build data processing and visualization pipelines through a visual programming (node-graph) interface.

- **License**: Simplified BSD (free for commercial use)
- **Current version**: see the root `CMakeLists.txt` (project declared via `project(${IVW_CFG_PROJECT_NAME} ... VERSION ...)`, with the canonical version string assembled as `IVW_VERSION` from `IVW_MAJOR_VERSION`, `IVW_MINOR_VERSION`, etc.)
- **Cross Platform**: Windows, Linux, macOS
- **Main website**: https://inviwo.org
- **API docs**: https://inviwo.org/inviwo/doc
- **GitHub**: https://github.com/inviwo/inviwo
---

## Repository Layout

```
inviwo/
├── .github/              # GitHub Actions workflows, issue/PR templates
├── apps/                 # Executable applications (inviwo GUI, inviwopyapp, minimal examples)
├── cmake/                # Reusable CMake helper functions and macros
├── data/                 # Example datasets (volumes, meshes, transfer functions)
├── docs/                 # Sphinx/Doxygen documentation sources, style guides
├── ext/                  # Vendored external dependencies (flags, ticpp)
├── include/inviwo/       # Public C++ headers for the core framework
│   ├── core/             # Framework core: processors, data structures, properties, algorithms
│   └── qt/               # Qt-based GUI: network editor, properties panel, canvas
├── modules/              # 40+ extension modules (rendering, I/O, UI, Python, etc.)
├── resources/            # Application resources (icons, images, stylesheets)
├── src/                  # Source files for the core framework (mainly for internal implementation)
├── tests/                # Unit, integration, regression tests, and benchmarks
├── tools/
│   └── meta/             # Code-generation tool for scaffolding modules and processors
├── CMakeLists.txt        # Root CMake configuration
├── CMakePresets.json     # Named build presets
├── vcpkg.json            # vcpkg dependency manifest
├── .clang-format         # C++ code formatting rules
├── .clang-tidy           # Static analysis configuration
└── .editorconfig         # Editor settings
```

---

## Build System

### Requirements
- **CMake** ≥ 4.0
- **vcpkg** for dependency management (bootstrapped automatically via CMake presets)
- **Compiler**: MSVC 2026 (Windows), Clang (macOS), GCC or Clang (Linux)
- **Qt 6** (required for GUI applications)
- **Python 3** (required for Python bindings)

### Key CMake Presets (`CMakePresets.json`)

| Preset | Purpose |
|--------|---------|
| `user` | Standard release build, shared libs, basic apps |
| `developer` | Full development: assertions, profiling, all tests, all apps |
| `msvc-developer` | Windows MSVC developer build |
| `xcode-developer` | macOS Xcode developer build |
| `ninja-developer` | Ninja-based developer build (Linux/macOS) |
| `gha-dynamic` / `gha-static` | CI presets used by GitHub Actions defined in `.github/presets/*.json`|

### Building

```bash
# Clone
git clone https://github.com/inviwo/inviwo
git clone https://github.com/microsoft/vcpkg

# Configure
cmake -S inviwo --preset ninja-developer

# Build all targets
cmake --build builds/ninja-developer --parallel

# Run tests
ctest --test-dir builds/ninja-developer --output-on-failure

# Build a single module target
cmake --build builds/ninja-developer --target inviwo-module-base
```

### Common CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `IVW_ENABLE_PYTHON` | ON | Python 3 bindings via pybind11 |
| `IVW_ENABLE_QT` | ON | Qt6-based GUI |
| `BUILD_SHARED_LIBS` | ON | Shared vs. static linking |
| `IVW_TEST_UNIT_TESTS` | ON | GTest unit tests |
| `IVW_TEST_INTEGRATION_TESTS` | ON | Integration tests (requires GLFW + Base module) |
| `IVW_TEST_BENCHMARKS` | OFF | Google Benchmark tests |
| `IVW_CFG_PROFILING` | OFF | Tracy/profiling instrumentation |
| `IVW_EXTERNAL_MODULES` | - | Semicolon-separated paths to extra modules |

---

## Code Style & Conventions

### C++ Style (enforced by `.clang-format`)
- **Based on**: Google C++ style with customizations
- **Indentation**: 4 spaces (no tabs)
- **Line length**: 100 characters
- **Pointer alignment**: Left — `int* ptr;` not `int *ptr;`
- **Qualifier alignment**: Left — `const int` not `int const`
- **Constructor initializers**: Break before comma
- `SortIncludes: false` — manage include order manually

Run formatter: `clang-format -i <file>`

### General Conventions
- UTF-8 source files with Unix line endings
- Trailing whitespace trimmed, final newline required (see `.editorconfig`)
- Use C++23 features freely (structured bindings, `if constexpr`, `std::optional`, etc.)
- Prefer `std::string_view` and `std::span` over raw pointers
- Use `fmt::format` (bundled) for string formatting, not `std::sprintf`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inviwo/inviwo](https://github.com/inviwo/inviwo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
