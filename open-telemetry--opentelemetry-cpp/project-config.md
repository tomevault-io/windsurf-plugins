---
trigger: always_on
description: OpenTelemetry C++ is a comprehensive telemetry SDK providing APIs and
---

# OpenTelemetry C++

OpenTelemetry C++ is a comprehensive telemetry SDK providing APIs and
implementations for traces, metrics, and logs. It supports both CMake and Bazel
build systems and runs on Linux, macOS, and Windows with modern C++ compilers
(C++14/17/20).

Always reference these instructions first and fallback to search or bash
commands only when you encounter unexpected information that does not match the
info here.

## Working Effectively

### Bootstrap, Build, and Test the Repository

**CRITICAL: NEVER CANCEL builds or long-running commands. Set appropriate
timeouts.**

#### CMake Build (Recommended for Development)

```bash
# Install basic dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y build-essential cmake git pkg-config

# Configure CMake build
mkdir -p build && cd build
cmake ..
# Takes ~12 seconds. Always completes quickly.

# Build the project
make -j$(nproc)
# Takes ~3 minutes. NEVER CANCEL. Set timeout to 15+ minutes.

# Run all tests
ctest --output-on-failure
# Takes ~24 seconds. Set timeout to 5+ minutes.
```

#### CI Build (Full Validation)

```bash
# Run the full CI validation (includes additional exporters)
./ci/do_ci.sh cmake.test
# Takes ~5.2 minutes. NEVER CANCEL. Set timeout to 20+ minutes.
```

#### Bazel Build (Alternative)

```bash
# Install bazelisk (managed Bazel)
sudo ./ci/install_bazelisk.sh

# Build simple example
bazel build //examples/simple:example_simple
# Time varies. NEVER CANCEL. Set timeout to 15+ minutes.

# Run simple example
bazel-bin/examples/simple/example_simple
```

**Note**: Bazel may have network connectivity issues in some environments when
downloading the required Bazel version (7.1.1).

### Validation

Always validate your changes using these steps after making code modifications:

#### Core Validation Scenario

```bash
# 1. Build and test successfully
cd build && make -j$(nproc) && ctest --output-on-failure

# 2. Run a simple example to verify functionality
./examples/simple/example_simple
# Should output telemetry spans with service.name, trace_id, span_id

# 3. Format code properly
./tools/format.sh
# Takes ~30 seconds. Must complete without errors.

# 4. Validate with maintainer mode (CRITICAL for warnings)
./ci/do_ci.sh cmake.maintainer.sync.test
# Takes ~4-6 minutes. NEVER CANCEL. Ensures all warnings are resolved.
```

#### Required Tools for Formatting

```bash
# Install formatting dependencies
pip install cmake_format                    # For CMake files
go install github.com/bazelbuild/buildtools/buildifier@latest  # For Bazel files
# clang-format should already be available on most systems
```

### Maintainer Mode Validation

**CRITICAL**: Always run maintainer mode builds to ensure warning-free code:

```bash
# Run maintainer mode validation
./ci/do_ci.sh cmake.maintainer.sync.test

# What this does:
# - Enables -Wall -Werror -Wextra compiler flags
# - Treats all warnings as errors
# - Ensures strict code quality standards
# - Required for all contributions
```

Maintainer mode (`-DOTELCPP_MAINTAINER_MODE=ON`) is essential for catching potential issues that would cause CI failures. It enables the strictest warning levels and treats warnings as compilation errors.

### CI Integration

Always run these before committing to ensure CI will pass:

```bash
# Format all code
./tools/format.sh

# Run linting (if shellcheck available for shell scripts)
shellcheck --severity=error ci/*.sh

# CRITICAL: Validate with maintainer mode to catch all warnings
./ci/do_ci.sh cmake.maintainer.sync.test
# Takes ~4-6 minutes. Enables -Wall -Werror -Wextra for strict validation.

# Validate build with additional exporters
./ci/do_ci.sh cmake.test
```

## Common Tasks

### Building and Running Examples

Examples demonstrate OpenTelemetry functionality and validate your environment:

```bash
# Build and run simple tracing example
cd build
make example_simple
./examples/simple/example_simple

# Build and run logs example
make logs_simple_example
./examples/logs_simple/logs_simple_example

# Build and run batch processing example
make batch_example
./examples/batch/batch_example
```

### Testing Changes

```bash
# Run specific test groups
ctest -R trace                    # Run only trace tests
ctest -R metrics                  # Run only metrics tests
ctest -R logs                     # Run only logs tests

# Run tests with verbose output for debugging
ctest --verbose --output-on-failure

# Run a specific test by name
ctest -R "trace.SystemTimestampTest.Construction" --verbose
```

### Key Directories and Navigation

```text
api/                  - Public OpenTelemetry API headers
sdk/                  - SDK implementation (most business logic)
exporters/            - Output plugins (ostream, memory, etc.)
examples/             - Sample applications demonstrating usage
  ├── simple/         - Basic tracing example (start here)
  ├── logs_simple/    - Basic logging example
  ├── metrics_simple/ - Basic metrics example (may be disabled)
  └── batch/          - Batch processing example
ci/                   - CI scripts and build automation
tools/                - Development tools (formatting, etc.)
test_common/          - Shared test utilities
third_party/          - External dependencies
```

### Important Files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-telemetry/opentelemetry-cpp](https://github.com/open-telemetry/opentelemetry-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
