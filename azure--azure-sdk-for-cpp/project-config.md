---
trigger: always_on
description: This file provides guidance for AI agents (e.g., GitHub Copilot, LLM-based assistants, and automation tools) interacting with the Azure SDK for C++ repository.
---

# AGENTS.md

This file provides guidance for AI agents (e.g., GitHub Copilot, LLM-based assistants, and automation tools) interacting with the Azure SDK for C++ repository.

## Repository Purpose and Scope

The **Azure SDK for C++** provides modern C++ libraries for Azure services, following the [Azure SDK Design Guidelines for C++](https://azure.github.io/azure-sdk/cpp_introduction.html). This repository contains:

- **Client libraries** for Azure services (Storage, Key Vault, Identity, etc.)
- **Core libraries** (`azure-core`) providing shared functionality (HTTP pipeline, authentication, retry policies, etc.)
- **Build infrastructure** using CMake and vcpkg
- **Documentation** and samples for developers
- **Test infrastructure** including unit tests and integration tests

### Key Technologies
- **Language**: C++14
- **Build System**: CMake (version 3.13+)
- **Package Manager**: vcpkg (manifest mode)
- **Testing**: Google Test (gtest)
- **Documentation**: Doxygen
- **CI/CD**: Azure Pipelines

## Agent Capabilities and Boundaries

### Supported Agent Actions

AI agents working in this repository are encouraged to:

1. **Code Contributions**
   - Implement new features following the [C++ Guidelines](https://azure.github.io/azure-sdk/cpp_introduction.html)
   - Fix bugs with minimal, surgical changes
   - Add or update unit tests using Google Test framework
   - Improve code documentation using Doxygen-style comments

2. **Documentation**
   - Update README files
   - Improve inline code comments
   - Update CONTRIBUTING.md when development processes change
   - Generate or update API documentation

3. **Build and Test**
   - Run CMake builds locally
   - Execute unit tests via `ctest`
   - Generate code coverage reports (when BUILD_CODE_COVERAGE is ON)
   - Build samples (when BUILD_SAMPLES is ON)

4. **Code Review and Analysis**
   - Review pull requests for adherence to guidelines
   - Suggest improvements to code structure and style
   - Identify potential issues before CI runs

### Actions Requiring Caution

1. **Dependency Management**
   - Changing vcpkg.json should be done sparingly
   - New dependencies require justification and approval
   - OpenSSL version changes need careful consideration
   - Consult `vcpkg-custom-ports` for custom dependency versions

2. **Breaking Changes**
   - Public API changes must follow semver and SDK guidelines
   - Internal types (in `_internal` namespace) can change within constraints
   - Private types (in `_detail` namespace) have fewer restrictions

3. **CI/CD Configuration**
   - Changes to `.github/workflows/` require testing
   - Azure Pipeline configurations need validation
   - Test proxy configuration changes need verification

### Actions Outside Agent Scope

Agents should **NOT**:

1. **Modify Security-Critical Code** without explicit review
   - Authentication flows
   - Credential handling
   - Cryptographic operations

2. **Make Large Architectural Changes** without design review
   - Changing core abstractions
   - Modifying HTTP pipeline architecture
   - Restructuring repository layout

3. **Commit Secrets or Sensitive Data**
   - No credentials in code or config files
   - No API keys in tests (use environment variables)
   - No connection strings in source

## Key Workflows and Commands

### Building the Project

```bash
# Basic build
mkdir build && cd build
cmake ..
cmake --build .

# Build with tests
cmake -DBUILD_TESTING=ON ..
cmake --build .

# Build with samples
cmake -DBUILD_SAMPLES=ON ..
cmake --build .

# Build with specific transport adapter
cmake -DBUILD_TRANSPORT_CURL=ON ..
cmake --build .
```

### Running Tests

```bash
# Set test mode (LIVE, PLAYBACK, or RECORD)
export AZURE_TEST_MODE=PLAYBACK

# Run all tests
ctest -V

# Run specific package tests
ctest -R azure-core
ctest -R storage

# Run with test proxy (for recording/playback)
export AZURE_TEST_USE_TEST_PROXY=ON
ctest -V
```

### Code Coverage

```bash
# Build with coverage enabled (requires Debug mode and GNU compiler)
cmake -DBUILD_TESTING=ON -DCMAKE_BUILD_TYPE=Debug -DBUILD_CODE_COVERAGE=ON ..
cmake --build .

# Generate coverage reports
make azure-core_cov_xml    # XML report
make azure-core_cov_html   # HTML report
```

### Documentation Generation

```bash
# Build Doxygen documentation
cmake -DBUILD_DOCUMENTATION=ON ..
cmake --build .
```

### Code Formatting

```bash
# Format code using clang-format
# The repository includes a .clang-format configuration
find sdk/ -name "*.cpp" -o -name "*.hpp" | xargs clang-format -i
```

## Repository Structure

```
azure-sdk-for-cpp/
+-- .github/              # GitHub configuration and workflows
    +-- copilot-instructions.md  # Copilot-specific guidance
    +-- workflows/        # CI/CD workflows
+-- cmake-modules/        # CMake helper modules
+-- doc/                  # Documentation
+-- eng/                  # Engineering system scripts
    +-- common/           # Shared scripts across Azure SDKs
    +-- docs/             # Documentation generation
+-- samples/              # Sample applications
+-- sdk/                  # SDK service libraries
    +-- core/             # Core libraries (azure-core, azure-core-amqp, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure/azure-sdk-for-cpp](https://github.com/Azure/azure-sdk-for-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
