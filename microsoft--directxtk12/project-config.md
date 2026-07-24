---
trigger: always_on
description: These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.
---

# GitHub Copilot Instructions

These instructions define how GitHub Copilot should assist with this project. The goal is to ensure consistent, high-quality code generation aligned with our conventions, stack, and best practices.

## Context

- **Project Type**: Graphics Library / DirectX / Direct3D 12 / Game Audio
- **Project Name**: DirectX Tool Kit for DirectX 12
- **Language**: C++
- **Framework / Libraries**: STL / CMake / CTest
- **Architecture**: Modular / RAII / OOP

## Getting Started

- See the tutorial at [Getting Started](https://github.com/microsoft/DirectXTK12/wiki/Getting-Started).
- The recommended way to integrate *DirectX Tool Kit for DirectX 12* into your project is by using the *vcpkg* Package Manager. See [d3d12game_vcpkg](https://github.com/walbourn/directx-vs-templates/tree/main/d3d12game_vcpkg) for a template which uses VCPKG.
- You can make use of the nuget.org packages **directxtk12_desktop_win10** or **directxtk12_uwp**.
- You can also use the library source code directly in your project or as a git submodule.

> If you are new to DirectX, you may want to start with [DirectX Tool Kit for DirectX 11](https://github.com/microsoft/DirectXTK/wiki/Getting-Started) to learn many important concepts for Direct3D programming, HLSL shaders, and the code patterns used in this project with a more 'newbie friendly' API.

## General Guidelines

- **Code Style**: The project uses an .editorconfig file to enforce coding standards. Follow the rules defined in `.editorconfig` for indentation, line endings, and other formatting. Additional information can be found on the wiki at [Implementation](https://github.com/microsoft/DirectXTK12/wiki/Implementation). The library's public API requires C++11, and the project builds with C++17 (`CMAKE_CXX_STANDARD 17`). This code is designed to build with Visual Studio 2022, Visual Studio 2026, clang for Windows v12 or later, or MinGW 12.2.
> Notable `.editorconfig` rules: C/C++ and HLSL files use 4-space indentation, `crlf` line endings, and `latin1` charset — avoid non-ASCII characters in source files. HLSL files have separate indent/spacing rules defined in `.editorconfig`.
- **Documentation**: The project provides documentation in the form of wiki pages available at [Documentation](https://github.com/microsoft/DirectXTK12/wiki/). The audio, input, and math implementations are identical to the DirectX Tool Kit for DirectX 11.
- **Error Handling**: Use C++ exceptions for error handling and use RAII smart pointers to ensure resources are properly managed. For some functions that return HRESULT error codes, they are marked `noexcept`, use `std::nothrow` for memory allocation, and should not throw exceptions.
- **Testing**: Unit tests for this project are implemented in this repository [Test Suite](https://github.com/walbourn/directxtk12test/) and can be run using CTest per the instructions at [Test Documentation](https://github.com/walbourn/directxtk12test/wiki). See [test copilot instructions](https://github.com/walbourn/directxtk12test/blob/main/.github/copilot-instructions.md) for additional information on the tests.
- **Security**: This project uses secure coding practices from the Microsoft Secure Coding Guidelines, and is subject to the `SECURITY.md` file in the root of the repository. Functions that read input from image files, geometry files, and audio files are subject to OneFuzz fuzz testing to ensure they are secure against malformed files.
- **Dependencies**: The project uses CMake and VCPKG for managing dependencies, making optional use of DirectXMath, DirectX-Headers, DirectX 12 Agility SDK, GameInput, and XAudio2Redist. The project can be built without these dependencies, relying on the Windows SDK for core functionality. CMake build options include `BUILD_GAMEINPUT`, `BUILD_WGI`, and `BUILD_XINPUT` for alternative input backends. Additional CMake build options include `BUILD_MIXED_DX11` for DX11 toolkit interop which removes shared code from the DX12 library as it will be present in the DX11 library to avoid link conflicts.
- **Continuous Integration**: This project implements GitHub Actions for continuous integration, ensuring that all code changes are tested and validated before merging. This includes building the project for a number of configurations and toolsets, running a subset of unit tests, and static code analysis including GitHub super-linter, CodeQL, and MSVC Code Analysis.
- **Code of Conduct**: The project adheres to the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). All contributors are expected to follow this code of conduct in all interactions related to the project.

## File Structure

```plaintext
.azuredevops/ # Azure DevOps pipeline configuration and policy files.
.github/      # GitHub Actions workflow files and linter configuration files.
.nuget/       # NuGet package configuration files.
build/        # Miscellaneous build files and scripts.
Audio/        # DirectX Tool Kit for Audio implementation files.
Inc/          # Public header files.
Src/          # Implementation header and source files.
  Shaders/    # HLSL shader files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/DirectXTK12](https://github.com/microsoft/DirectXTK12) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
