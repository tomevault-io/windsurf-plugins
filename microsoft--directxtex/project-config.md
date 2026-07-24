---
trigger: always_on
description: These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.
---

# GitHub Copilot Instructions

These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.

## Context

- **Project Type**: Graphics Library / DirectX / Direct3D 11 / Direct3D 12 / Image Processing
- **Project Name**: DirectXTex Texture Processing Library
- **Language**: C++
- **Framework / Libraries**: STL / CMake / CTest
- **Architecture**: Modular / RAII / OOP

## Getting Started

- See the tutorial at [Getting Started](https://github.com/microsoft/DirectXTex/wiki/Getting-Started).
- The recommended way to integrate *DirectXTex* into your project is by using the *vcpkg* Package Manager.
- You can make use of the nuget.org packages **directxtex_desktop_win10**, or **directxtex_uwp**.
- You can also use the library source code directly in your project or as a git submodule.

## General Guidelines

- **Code Style**: The project uses an .editorconfig file to enforce coding standards. Follow the rules defined in `.editorconfig` for indentation, line endings, and other formatting. Additional information can be found on the wiki at [Implementation](https://github.com/microsoft/DirectXTK/wiki/Implementation). The library's public API requires C++11, and the project builds with C++17 (`CMAKE_CXX_STANDARD 17`). The command-line tools also use C++17, including `<filesystem>` for long file path support. This code is designed to build with Visual Studio 2022, Visual Studio 2026, clang for Windows v12 or later, MinGW, and GCC on Linux.
> Notable `.editorconfig` rules: C/C++ and HLSL files use 4-space indentation, `crlf` line endings, and `latin1` charset — avoid non-ASCII characters in source files. HLSL files have separate indent/spacing rules defined in `.editorconfig`.
- **Documentation**: The project provides documentation in the form of wiki pages available at [Documentation](https://github.com/microsoft/DirectXTex/wiki/).
- **Error Handling**: Use C++ exceptions for error handling and uses RAII smart pointers to ensure resources are properly managed. For some functions that return HRESULT error codes, they are marked `noexcept`, use `std::nothrow` for memory allocation, and should not throw exceptions.
- **Testing**: Unit tests for this project are implemented in this repository [Test Suite](https://github.com/walbourn/directxtextest/) and can be run using CTest per the instructions at [Test Documentation](https://github.com/walbourn/directxtextest/wiki). See [test copilot instructions](https://github.com/walbourn/directxtextest/blob/main/.github/copilot-instructions.md) for additional information on the tests.
- **Security**: This project uses secure coding practices from the Microsoft Secure Coding Guidelines, and is subject to the `SECURITY.md` file in the root of the repository. Functions that read input from image files are subject to OneFuzz fuzz testing to ensure they are secure against malformed files.
- **Dependencies**: The project uses CMake and VCPKG for managing dependencies, making optional use of DirectXMath and DirectX-Headers. The project can be built without these dependencies, relying on the Windows SDK for core functionality.
- **Continuous Integration**: This project implements GitHub Actions for continuous integration, ensuring that all code changes are tested and validated before merging. Platforms tested include Windows (MSVC, clang-cl, MinGW), Windows on ARM64, Xbox, UWP, WSL (Linux with GCC), and macOS (Apple Clang). This includes building the project for a number of configurations and toolsets, running a subset of unit tests, and static code analysis including GitHub super-linter, CodeQL, and MSVC Code Analysis.
- **Code of Conduct**: The project adheres to the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). All contributors are expected to follow this code of conduct in all interactions related to the project.

## File Structure

```plaintext
.azuredevops/     # Azure DevOps pipeline configuration and policy files.
.github/          # GitHub Actions workflow files and linter configuration files.
.nuget/           # NuGet package configuration files.
build/            # Miscellaneous build files and scripts.
Auxiliary/        # Auxiliary functions such as Xbox tiling extensions, OpenEXR support, etc.
Common/           # Common utility functions shared between the library and command-line tools.
DirectXTex/       # DirectXTex implementation files.
  Shaders/        # HLSL shader files.
DDSView/          # Sample application for viewing DDS texture files using DirectXTex.
Texassemble/      # CLI tool for creating complex DDS files from multiple image files.
Texconv/          # CLI tool for converting image files to DDS texture files including block compression, mipmaps, and resizing.
Texdiag/          # CLI tool for diagnosing and validating DDS texture files.
DDSTextureLoader/ # Standalone version of the DDS texture loader for Direct3D 9/11/12.
ScreenGrab/       # Standalone version of the screenshot capture utility for Direct3D 9/11/12.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/DirectXTex](https://github.com/microsoft/DirectXTex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
