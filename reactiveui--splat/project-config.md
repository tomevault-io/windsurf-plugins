---
trigger: always_on
description: ﻿# Splat: Cross-Platform Library for .NET
---

﻿# Splat: Cross-Platform Library for .NET

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- **CRITICAL**: Requires .NET 9.0 SDK (not .NET 8.0). Install with:
  ```bash
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 9.0
  export PATH="$HOME/.dotnet:$PATH"
  ```
- **Platform Support**: This project builds on Windows, Linux, and macOS with cross-platform support.
- **Development Tools**: Visual Studio 2022, VS Code with C# extension, or JetBrains Rider.
- **Note on Cloning the Repository**:
  When cloning the Splat repository, use a full clone instead of a shallow one (e.g., avoid --depth=1). This project uses Nerdbank.GitVersioning for automatic version calculation based on Git history. Shallow clones lack the necessary commit history, which can cause build errors or force the tool to perform an extra fetch step to deepen the repository. To ensure smooth builds:
   ```bash
   git clone https://github.com/reactiveui/splat.git
   ```
  If you've already done a shallow clone, deepen it with:
   ```bash
   git fetch --unshallow
   ```
  This prevents exceptions like "Shallow clone lacks the objects required to calculate version height."

### Development Workflow
- Full solution restore and build:
  ```bash
  cd src
  dotnet restore Splat.sln
  dotnet build Splat.sln --configuration Release
  ```
  Build time: **2-5 minutes**. Set timeout to 10+ minutes for full solution builds.

- Individual project builds (faster for development):
  ```bash
  cd src
  dotnet build Splat/Splat.csproj --configuration Release
  dotnet build Splat.Tests/Splat.Tests.csproj --configuration Release
  ```

### Testing
- **Full test suite**:
  ```bash
  cd src
  dotnet test --configuration Release
  ```
  Test time: **2-5 minutes**. Set timeout to 15+ minutes.

- **Individual project tests** (faster for development):
  ```bash
  cd src
  dotnet test Splat.Tests/Splat.Tests.csproj --configuration Release
  ```

## Validation and Quality Assurance

### Code Style and Analysis Enforcement
- **EditorConfig Compliance**: Repository uses a comprehensive `.editorconfig` with detailed rules for C# formatting, naming conventions, and code analysis.
- **StyleCop Analyzers**: Enforces consistent C# code style with `stylecop.analyzers`.
- **Roslynator Analyzers**: Additional code quality rules with `Roslynator.Analyzers`.
- **Analysis Level**: Set to `latest` with enhanced .NET analyzers enabled.
- **CRITICAL**: All code must comply with ReactiveUI contribution guidelines: [https://www.reactiveui.net/contribute/index.html](https://www.reactiveui.net/contribute/index.html).

## C# Style Guide
**General Rule**: Follow "Visual Studio defaults" with the following specific requirements:

### Brace Style
- **Allman style braces**: Each brace begins on a new line.
- **Single line statement blocks**: Can go without braces but must be properly indented on its own line and not nested in other statement blocks that use braces.
- **Exception**: A `using` statement is permitted to be nested within another `using` statement by starting on the following line at the same indentation level, even if the nested `using` contains a controlled block.

### Indentation and Spacing
- **Indentation**: Four spaces (no tabs).
- **Spurious free spaces**: Avoid, e.g., `if (someVar == 0)...` where dots mark spurious spaces.
- **Empty lines**: Avoid more than one empty line at any time between members of a type.
- **Labels**: Indent one less than the current indentation (for `goto` statements).

### Field and Property Naming
- **Internal and private fields**: Use `_camelCase` prefix with `readonly` where possible.
- **Static fields**: `readonly` should come after `static` (e.g., `static readonly` not `readonly static`).
- **Public fields**: Use PascalCasing with no prefix (use sparingly).
- **Constants**: Use PascalCasing for all constant local variables and fields (except interop code, where names and values must match the interop code exactly).
- **Fields placement**: Specify fields at the top within type declarations.

### Visibility and Modifiers
- **Always specify visibility**: Even if it's the default (e.g., `private string _foo` not `string _foo`).
- **Visibility first**: Should be the first modifier (e.g., `public abstract` not `abstract public`).
- **Modifier order**: `public`, `private`, `protected`, `internal`, `static`, `extern`, `new`, `virtual`, `abstract`, `sealed`, `override`, `readonly`, `unsafe`, `volatile`, `async`.

### Namespace and Using Statements
- **Namespace imports**: At the top of the file, outside of `namespace` declarations.
- **Sorting**: System namespaces alphabetically first, then third-party namespaces alphabetically.
- **Global using directives**: Use where appropriate to reduce repetition across files.
- **Placement**: Use `using` directives outside `namespace` declarations.

### Type Usage and Variables
- **Language keywords**: Use instead of BCL types (e.g., `int`, `string`, `float` instead of `Int32`, `String`, `Single`) for type references and method calls (e.g., `int.Parse` instead of `Int32.Parse`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactiveui/splat](https://github.com/reactiveui/splat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
