---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Akavache: Asynchronous Key-Value Store

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- **CRITICAL**: Requires .NET 10, 9, and 8 SDKs. Always install all versions fresh (don't check what's installed):
  ```bash
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 10.0
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 9.0
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 8.0
  export PATH="$HOME/.dotnet:$PATH"
  ```
- **Platform Support**: This project has cross-platform support with proper setup. Windows has full support; Linux/macOS have comprehensive support for core libraries, Android, and MAUI development.
- **Development Tools**: Visual Studio 2022 or VS Code with C# extension.
- Note on Cloning the Repository
  When cloning the Akavache repository, use a full clone instead of a shallow one (e.g., avoid --depth=1). This project uses Nerdbank.GitVersioning for automatic version calculation based on Git history. Shallow clones lack the necessary commit history, which can cause build errors or force the tool to perform an extra fetch step to deepen the repository. To ensure smooth builds:
   ```bash
   git clone https://github.com/reactiveui/Akavache.git
   ```
   If you've already done a shallow clone, deepen it with:
   ```bash
   git fetch --unshallow
   ```
   This prevents exceptions like "Shallow clone lacks the objects required to calculate version height."

### Windows Development (Full Support)
- Install .NET workloads for cross-platform development:
  ```bash
  dotnet workload install android ios tvos macos maui maccatalyst
  ```
- Full solution restore and build:
  ```bash
  cd src
  dotnet build Akavache.slnx
  ```

### Linux/macOS Development (Comprehensive Support)
- **CRITICAL**: Install .NET 10, 9, and 8 SDKs first, then install required workloads:
  ```bash
  # Install .NET SDKs (all versions fresh)
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 10.0
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 9.0
  curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version latest --channel 8.0
  export PATH="$HOME/.dotnet:$PATH"

  # Install Android and MAUI workloads (recommended for cross-platform development)
  dotnet workload install android maui-android
  ```
- **What works on Linux/macOS**:
  - All core libraries (Akavache.Core, Akavache.SystemTextJson, etc.)
  - Android projects (net9.0-android, net10.0-android) - requires Android workloads
  - MAUI applications - samples build successfully
  - All test assemblies (`dotnet test` from src/)
- **What fails on Linux/macOS**:
  - Windows-specific projects (WPF samples, net9.0-windows)
  - .NET Framework projects (net462, net472, net481)
- **Building individual projects**: Always use explicit targeting when needed:
  ```bash
  cd src
  dotnet build Akavache.Core/Akavache.csproj -p:TargetFramework=net10.0
  ```

### Testing
- **CRITICAL**: Test execution requires .NET SDKs (10, 9, and 8).
- The solution uses **slnx** format: `Akavache.slnx`
- Test commands — **always run from `src/` directory**:
  ```bash
  cd src

  # Run a specific test assembly
  dotnet test --project tests/Akavache.Core.Tests/Akavache.Core.Tests.csproj
  dotnet test --project tests/Akavache.Sqlite3.Tests/Akavache.Sqlite3.Tests.csproj
  dotnet test --project tests/Akavache.EncryptedSqlite3.Tests/Akavache.EncryptedSqlite3.Tests.csproj
  dotnet test --project tests/Akavache.Integration.Tests/Akavache.Integration.Tests.csproj
  dotnet test --project tests/Akavache.Settings.Tests/Akavache.Settings.Tests.csproj
  dotnet test --project tests/Akavache.HttpDownloader.Tests/Akavache.HttpDownloader.Tests.csproj

  # Parallel assemblies
  dotnet test --project tests/Akavache.Core.Tests.Parallel/Akavache.Core.Tests.Parallel.csproj
  dotnet test --project tests/Akavache.Sqlite3.Tests.Parallel/Akavache.Sqlite3.Tests.Parallel.csproj
  dotnet test --project tests/Akavache.EncryptedSqlite3.Tests.Parallel/Akavache.EncryptedSqlite3.Tests.Parallel.csproj
  dotnet test --project tests/Akavache.Integration.Tests.Parallel/Akavache.Integration.Tests.Parallel.csproj
  dotnet test --project tests/Akavache.Settings.Tests.Parallel/Akavache.Settings.Tests.Parallel.csproj
  ```

### Test Architecture

Tests are split into **serial** and **parallel** assemblies:

- **Serial assemblies** (`*.Tests`): Use `[assembly: NotInParallel]` with a custom `AkavacheTestExecutor` that resets global state (`CacheDatabase`, `AppLocator`, `UniversalSerializer`) between tests. Tests that touch shared singletons live here.
- **Parallel assemblies** (`*.Tests.Parallel`): No executor, TUnit default parallel execution. Tests that create isolated cache instances and don't touch global state live here.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reactiveui/Akavache](https://github.com/reactiveui/Akavache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
