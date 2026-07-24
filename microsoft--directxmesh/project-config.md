---
trigger: always_on
description: These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.
---

# GitHub Copilot Instructions

These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.

## Context

- **Project Type**: Graphics Library / DirectX / Direct3D 11 / Direct3D 12 / Geometry Processing
- **Project Name**: DirectXMesh Geometry Processing Library
- **Language**: C++
- **Framework / Libraries**: STL / CMake / CTest
- **Architecture**: Modular / RAII / OOP

## Getting Started

- See the tutorial at [Getting Started](https://github.com/microsoft/DirectXMesh/wiki/Getting-Started).
- The recommended way to integrate *DirectXMesh* into your project is by using the *vcpkg* Package Manager.
- You can make use of the nuget.org packages **directxmesh_desktop_win10** or **directxmesh_uwp**.
- You can also use the library source code directly in your project or as a git submodule.

## General Guidelines

- **Code Style**: The project uses an .editorconfig file to enforce coding standards. Follow the rules defined in `.editorconfig` for indentation, line endings, and other formatting. Additional information can be found on the wiki at [Implementation](https://github.com/microsoft/DirectXMesh/wiki/Implementation). The library's public API requires C++11, and the project builds with C++17 (`CMAKE_CXX_STANDARD 17`). The command-line tools also use C++17, including `<filesystem>` for long file path support. This code is designed to build with Visual Studio 2022, Visual Studio 2026, clang for Windows v12 or later, MinGW, and GCC on Linux.
> Notable `.editorconfig` rules: C/C++ files use 4-space indentation, `crlf` line endings, and `latin1` charset — avoid non-ASCII characters in source files.
- **Documentation**: The project provides documentation in the form of wiki pages available at [Documentation](https://github.com/microsoft/DirectXMesh/wiki/).
- **Error Handling**: Use C++ exceptions for error handling and uses RAII smart pointers to ensure resources are properly managed. For some functions that return HRESULT error codes, they are marked `noexcept`, use `std::nothrow` for memory allocation, and should not throw exceptions.
- **Testing**: Unit tests for this project are implemented in this repository [Test Suite](https://github.com/walbourn/directxmeshtest/) and can be run using CTest per the instructions at [Test Documentation](https://github.com/walbourn/directxmeshtest/wiki). See [test copilot instructions](https://github.com/walbourn/directxmeshtest/blob/main/.github/copilot-instructions.md) for additional information on the tests.
- **Security**: This project uses secure coding practices from the Microsoft Secure Coding Guidelines, and is subject to the `SECURITY.md` file in the root of the repository. Functions that read input from geometry files are subject to OneFuzz fuzz testing to ensure they are secure against malformed files.
- **Dependencies**: The project uses CMake and VCPKG for managing dependencies, making optional use of DirectXMath and DirectX-Headers. The project can be built without these dependencies, relying on the Windows SDK for core functionality.
- **Continuous Integration**: This project implements GitHub Actions for continuous integration, ensuring that all code changes are tested and validated before merging. Platforms tested include Windows (MSVC, clang-cl, MinGW), Windows on ARM64, Xbox, UWP, WSL (Linux with GCC), and macOS (Apple Clang). This includes building the project for a number of configurations and toolsets, running a subset of unit tests, and static code analysis including GitHub super-linter, CodeQL, and MSVC Code Analysis.
- **Code of Conduct**: The project adheres to the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). All contributors are expected to follow this code of conduct in all interactions related to the project.

## File Structure

```plaintext
.azuredevops/ # Azure DevOps pipeline configuration and policy files.
.github/      # GitHub Actions workflow files and linter configuration files.
.nuget/       # NuGet package configuration files.
build/        # Miscellaneous build files and scripts.
DirectXMesh/  # DirectXMesh implementation files.
Utilities/    # Utility headers such as a WaveFront .obj file loader and a FVF converter.
Meshconvert/  # CLI tool for importing WaveFront .obj files and converting to CMO, SDKMESH, or VBO formats.
skills/       # Published CoPilot skills for use by developers.
Tests/        # Tests are designed to be cloned from a separate repository at this location.
wiki/         # Local clone of the GitHub wiki documentation repository.
```

## Patterns

### Patterns to Follow

- Use RAII for all resource ownership (memory, file handles, etc.).
- Many classes utilize the pImpl idiom to hide implementation details, and to enable optimized memory alignment in the implementation.
- Use `std::unique_ptr` for exclusive ownership.
- Make use of anonymous namespaces to limit scope of functions and variables.
- Make use of `assert` for debugging checks, but be sure to validate input parameters in release builds.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/DirectXMesh](https://github.com/microsoft/DirectXMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
