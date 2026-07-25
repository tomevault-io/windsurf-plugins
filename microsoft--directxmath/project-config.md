---
trigger: always_on
description: These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.
---

# GitHub Copilot Instructions

These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.

## Context

- **Project Type**: Math Library / DirectX / Direct3D
- **Project Name**: DirectXMath SIMD C++ linear algebra library
- **Language**: C++ (minimum C++11; C++14, C++17, and C++20 features used conditionally)
- **Framework / Libraries**: STL / CMake / CTest
- **Compiler Requirement**: Visual C++ 2019 (16.11), Visual Studio 2022, or Visual Studio 2026 or later when using MSVC.

## Getting Started

- See the Getting Started guide on [Microsoft Learn](https://learn.microsoft.com/windows/win32/dxmath/pg-xnamath-getting-started).
- The recommended way to integrate *DirectXMath* into your project is by using the *vcpkg* Package Manager.
- You can make use of the nuget.org package **directxmath**.
- You can also use the library source code directly in your project or as a git submodule.
- The Windows SDK includes the DirectXMath library, although that version is not as up-to-date as other integration methods.

## General Guidelines

- **Code Style**: The project uses an .editorconfig file to enforce coding standards. Follow the rules defined in `.editorconfig` for indentation, line endings, and other formatting. Additional information can be found on the wiki at [Implementation](https://github.com/microsoft/DirectXMath/wiki/Implementation). The code requires C++11/C++14 features.
> Notable `.editorconfig` rules: C/C++ files use 4-space indentation, `crlf` line endings, and `latin1` charset — avoid non-ASCII characters in source files.
- **Documentation**: The project provides documentation on [Microsoft Learn](https://learn.microsoft.com/windows/win32/dxmath/directxmath-portal) with additional wiki pages available on [GitHub](https://github.com/microsoft/DirectXMath/wiki/). The project does **not** use Doxygen.
- **Error Handling**: The majority of functions have no error conditions and do not throw C++ exceptions which is why they are marked `noexcept`. A few functions have `bool` results to indicate success or failure.
- **Testing**: Unit tests for this project are implemented in a separate repository [Test Suite](https://github.com/walbourn/directxmathtest/) and can be run using CTest per the instructions at [Test Documentation](https://github.com/walbourn/directxmathtest/wiki). See [test copilot instructions](https://github.com/walbourn/directxmathtest/blob/main/.github/copilot-instructions.md) for additional information on the tests.
- **Security**: This project uses secure coding practices from the Microsoft Secure Coding Guidelines, and is subject to the `SECURITY.md` file in the root of the repository.
- **Dependencies**: The project has minimal dependencies, primarily relying on compiler intrinsics. It is designed to be self-contained and portable across different platforms and toolsets.
- **Continuous Integration**: This project has 20 GitHub Actions workflows covering MSVC, Clang/LLVM, GCC (WSL), ARM64, macOS, Address Sanitizer, CodeQL, and super-linter. Workflows are in `.github/workflows/` and include compiler-specific builds (`msvc.yml`, `clangcl.yml`, `cxx.yml`), platform-specific builds (`arm64.yml`, `arm64test.yml`, `macos.yml`, `wsl.yml`, `wslcxx.yml`), extension tests (`shmath.yml`, `shmathclang.yml`, `xdsp.yml`, `xdspclang.yml`), and static analysis (`codeql.yml`, `lint.yml`, `asan.yml`). Additional workflows include `main.yml`, `msbuild.yml`, `msbuildex.yml`, and `test.yml`. Azure DevOps pipeline configurations are in `.azuredevops/`.
- **Code of Conduct**: The project adheres to the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). All contributors are expected to follow this code of conduct in all interactions related to the project.

## File Structure

```plaintext
.azuredevops/ # Azure DevOps pipeline configuration and policy files.
.github/      # GitHub Actions workflow files and linter configuration files.
.nuget/       # NuGet package configuration files.
build/        # Miscellaneous build files and scripts (CMake config templates, pkg-config, PowerShell helpers).
Inc/          # DirectXMath public and implementation files. The library is header-only, so all files are in this directory.
Extensions/   # Extensions to the DirectXMath library with standalone SSE-level specific functions for runtime selection of SIMD instruction set.
MatrixStack/  # D3DX9-like matrix stack implementation for DirectXMath.
SHMath/       # Spherical harmonic functions using DirectXMath.
Stereo3D/     # Stereo 3D projection matrix functions using DirectXMath created for HoloLens.
XDSP/         # Digital Signal Processing (DSP) functions using DirectXMath.
skills/       # Published CoPilot skills for use by developers.
Tests/        # Tests are designed to be cloned from a separate repository at this location.
wiki/         # Local clone of the GitHub wiki documentation repository.
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/DirectXMath](https://github.com/microsoft/DirectXMath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
