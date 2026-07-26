---
trigger: always_on
description: **DDS** is a double-dummy solver for bridge hands, written in C++. It calculates the maximum number of tricks a partnership can take for a given contract. The library:
---

# DDS (Double Dummy Solver) Repository Instructions

## Project Overview

**DDS** is a double-dummy solver for bridge hands, written in C++. It calculates the maximum number of tricks a partnership can take for a given contract. The library:

- Supports multi-threading (OpenMP, GCD, STL, PPL implementations)
- Works on Windows, Linux, and macOS
- Provides both C and C++ APIs
- Uses transposition tables for efficient solving
- Is actively being modernized to version 3.0 with improved memory management and C++20 features

**Current State**: Work in progress towards version 3.0. The project has been refactored from the legacy 2.9.0 codebase to use modern C++ features, Bazel build system, and modular architecture.

**Languages & Tools**: C++20, Bazel (version pinned in `.bazelversion`), GoogleTest for testing

**Repository Size**: ~50 C++ source files in library/src, ~20 test files, several examples

## Building and Testing

**ALWAYS use Bazel** for building and testing. Legacy Makefiles exist but are deprecated.

### Essential Commands

```bash
# Build everything (required before testing or running)
bazel build //...

# Run all tests
bazel test //...

# Build specific target
bazel build //library/src:dds

# Run specific test
bazel test //library/tests:dtest

# Build with optimization
bazel build -c opt //...

# Build with debug symbols
bazel build -c dbg //...

# Build with AddressSanitizer (see .bazelrc build:asan and docs/BUILD_SYSTEM.md)
bazel build --config=asan //...
```

### Build Time Expectations
(On a modern multi-core system with 8+ cores and SSD)
- Initial build (cold cache): 2-5 minutes
- Incremental builds: 10-30 seconds
- Full test suite: 1-3 minutes
- Individual test: 2-10 seconds

Note: Times may vary significantly based on hardware and network conditions.

### Before Making Changes
ALWAYS run these commands first to establish baseline:
```bash
bazel build //...
bazel test //...
```
This ensures any pre-existing issues are not attributed to your changes.

### After Making Changes
ALWAYS validate your changes:
```bash
# Build and test
bazel build //...
bazel test //...
```

## Project Layout and Architecture

### Directory Structure
```
<repository-root>/
├── .github/
│   ├── copilot-instructions.md     # This file
│   ├── instructions/                # Path-specific instructions
│   │   ├── cpp.instructions.md     # C++ style guide
│   │   ├── bazel.instructions.md   # Bazel build rules
│   │   ├── github.instructions.md  # PR/workflow rules
│   │   └── git.instructions.md     # Git usage
│   └── workflows/
│       ├── ci_linux.yml            # Linux CI pipeline
│       └── ci_macos.yml            # macOS CI pipeline
├── library/
│   ├── src/                        # Main source code
│   │   ├── api/                    # Public C/C++ API headers
│   │   │   ├── dds.h              # Internal header (data structures, included by dds.hpp)
│   │   │   ├── dll.h              # C API
│   │   │   └── solve_board.hpp    # C++ solver interface
│   │   ├── solver_context/         # Solver state management
│   │   ├── trans_table/            # Transposition table implementation
│   │   ├── system/                 # Threading/memory/system utilities
│   │   ├── moves/                  # Move generation
│   │   ├── heuristic_sorting/      # Move ordering heuristics
│   │   ├── lookup_tables/          # Precomputed tables
│   │   ├── utility/                # Helper functions
│   │   ├── ab_search.cpp/hpp       # Alpha-beta search core
│   │   ├── solve_board.cpp/hpp     # Main solve entry points
│   │   ├── par.cpp                 # Par score calculations
│   │   └── BUILD.bazel             # Build configuration
│   └── tests/                      # Test suites
│       ├── dtest.cpp               # Main test driver
│       ├── system/                 # System tests
│       └── regression/             # Regression tests
├── examples/                       # Example programs
│   ├── par.cpp
│   ├── solve_all_boards.cpp
│   └── BUILD.bazel
├── hands/                          # Test hand files
│   └── list100.txt                 # 100 test hands
├── docs/                           # Documentation
├── copilot/                        # Copilot-related files (plans, tasks)
├── BUILD.bazel                     # Root build file
├── MODULE.bazel                    # Bazel module definition
├── .bazelrc                        # Bazel configuration
├── .clang-tidy                     # C++ linter configuration
└── README.md
```

### Key Files to Know

**Build Configuration**:
- `BUILD.bazel` - Root build targets, exports `//library/src:dds` library
- `library/src/BUILD.bazel` - Main library build rules
- `MODULE.bazel` - External dependencies (GoogleTest, etc.)
- `.bazelrc` - Compiler flags, build modes

**Public API** (use these include paths):
- `#include <api/dll.h>` - C API
- `#include <dds/dds.hpp>` - C++ API (includes api/dll.h and api/solve_board.hpp)
  - Note: The file is at `library/src/dds.hpp`, but the `dds/` prefix is added by Bazel's `include_prefix` directive in the build rules

**Public API Files** (actual file locations):
- `library/src/api/dll.h` - C API declarations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dds-bridge/dds](https://github.com/dds-bridge/dds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
