---
trigger: always_on
description: qpdf is a command-line tool and C++ library that performs content-preserving transformations on
---

# Copilot Coding Agent Instructions for qpdf

## Repository Summary
qpdf is a command-line tool and C++ library that performs content-preserving transformations on
PDF files. It supports linearization, encryption, page splitting/merging, and PDF file inspection.
Version: 12.3.0.

**Project Type:** C++ library and CLI tool (C++20 standard)  
**Build System:** CMake 3.16+ with Ninja generator  
**External Dependencies:** zlib, libjpeg, OpenSSL, GnuTLS (crypto providers)

## Build Instructions

### Quick Build (Recommended)
```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get install build-essential cmake ninja-build zlib1g-dev libjpeg-dev libgnutls28-dev libssl-dev

# Configure and build
cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build -j$(nproc)

# Run tests
cd build && ctest --output-on-failure
```

### Using CMake Presets (Maintainer Mode)
```bash
cmake --preset maintainer          # Configure
cmake --build --preset maintainer  # Build
ctest --preset maintainer          # Test
```

Available presets: `maintainer`, `maintainer-debug`, `maintainer-coverage`, `maintainer-profile`,
`debug`, `release`, `sanitizers`, `msvc`, `msvc-release`. Use `cmake --list-presets` to see all
options.

### Build Notes
- **Always build out-of-source** in a subdirectory (e.g., `build/`). In-source builds are
  explicitly blocked.
- Build time: approximately 2-3 minutes on typical CI runners.
- Test suite time: approximately 1 minute for all 7 test groups.
- The `MAINTAINER_MODE` cmake option enables stricter checks and auto-generation of job files.

## Running Tests
```bash
cd build

# Run all tests
ctest --output-on-failure

# Run specific test groups
ctest -R qpdf        # Main qpdf CLI tests (~43 seconds)
ctest -R libtests    # Library unit tests (~8 seconds)
ctest -R examples    # Example code tests
ctest -R fuzz        # Fuzzer tests

# Run with verbose output
ctest --verbose
```

**Test Framework:** Tests use `qtest` (a Perl-based test framework). Tests are invoked via `ctest`
and compare outputs against expected files. Test coverage uses `QTC::TC` macros.

## Code Formatting
```bash
./format-code   # Formats all C/C++ files with clang-format
```
- Requires **clang-format version 20 or higher**.
- Configuration: `.clang-format` in the repository root.
- Always run before committing changes to C/C++ files.

## Project Layout

### Key Directories
| Directory | Purpose |
|-----------|---------|
| `libqpdf/` | Core library implementation (*.cc files) |
| `include/qpdf/` | Public headers (QPDF.hh, QPDFObjectHandle.hh, QPDFWriter.hh) |
| `qpdf/` | CLI executable and main test driver |
| `libtests/` | Library unit tests |
| `examples/` | Example programs demonstrating API usage |
| `fuzz/` | Fuzzer test programs for oss-fuzz |
| `manual/` | Documentation (reStructuredText for Sphinx) |
| `build-scripts/` | CI and build automation scripts |

### Important Files
| File | Purpose |
|------|---------|
| `CMakeLists.txt` | Main build configuration |
| `CMakePresets.json` | Predefined build configurations |
| `job.yml` | Command-line argument definitions (auto-generates code) |
| `generate_auto_job` | Python script that generates argument parsing code |
| `.clang-format` | Code formatting rules |
| `README-developer.md` | Developer guidelines for modifying qpdf |
| `README-maintainer.md` | Maintainer notes for release preparation and maintainers |

### Auto-Generated Files
When modifying `job.yml` or CLI options, regenerate with:
```bash
./generate_auto_job --generate
# Or build with: cmake -DGENERATE_AUTO_JOB=ON
```

## CI Workflows (.github/workflows/)

### main.yml (Primary CI)
- **Prebuild**: Documentation and external libs preparation
- **Linux**: Full build and test with image comparison
- **Windows**: MSVC and MinGW builds (32/64-bit)
- **macOS**: macOS build
- **AppImage**: Linux AppImage generation
- **Sanitizers**: AddressSanitizer and UndefinedBehaviorSanitizer tests
- **CodeCov**: Coverage reporting
- **pikepdf**: Compatibility testing with pikepdf Python library

## Coding Conventions

### Must Follow
1. **Assertions**: Test code should include `qpdf/assert_test.h` first. Debug code should include
   `qpdf/assert_debug.h` and use `qpdf_assert_debug` instead of `assert`. Use `qpdf_expect`,
   `qpdf_ensures`, `qpdf_invariant` for pre/post-conditions. Never use raw `assert()` in non-test
   code. The `check-assert` test enforces this.
2. **Use `QIntC` for type conversions** - Required for safe integer casting.
3. **Avoid `operator[]`** - Use `.at()` for std::string and std::vector (see README-hardening.md).
4. **Include order**: Include the class's own header first, then a blank line, then other includes.
5. **Use `std::to_string`** instead of QUtil::int_to_string.
6. **Preserve existing symbols by default**: Do not remove or rename existing functions, methods,
   constructors, enum values, or CLI option names unless explicitly requested in the task.
7. **Locale-safe number output** - Always imbue `ostringstream` with `std::locale::classic()`
   before outputting numbers, to prevent the user's global locale from altering numeric output.
8. **DLL export annotation** - New public methods in `include/qpdf/` must be annotated with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qpdf/qpdf](https://github.com/qpdf/qpdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
