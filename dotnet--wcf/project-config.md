---
trigger: always_on
description: This repository contains the .NET (Core) WCF **client** libraries. These are the client-side subset of Windows Communication Foundation, ported to .NET Core/.NET. WCF *service* hosting is in the separate [CoreWCF](https://github.com/CoreWCF/CoreWCF) project.
---

# Copilot Instructions for dotnet/wcf

This repository contains the .NET (Core) WCF **client** libraries. These are the client-side subset of Windows Communication Foundation, ported to .NET Core/.NET. WCF *service* hosting is in the separate [CoreWCF](https://github.com/CoreWCF/CoreWCF) project.

## Build and Test

This repo uses [Microsoft.DotNet.Arcade.Sdk](https://github.com/dotnet/arcade) for build infrastructure. After running `build -restore`, a repo-local dotnet SDK is installed at `.dotnet/` under the repository root.

### Build commands (run from repo root)

```powershell
# Restore + build (default action)
build.cmd

# Build without running tests
build.cmd -- /p:SkipTests=true

# Build in Release configuration
build.cmd -c Release
```

### Using dotnet CLI directly

After a successful `build -restore`, set these environment variables to use the repo-local SDK (replace `<repo_root>` with your clone path):

```powershell
$env:DOTNET_ROOT = '<repo_root>/.dotnet/'
$env:DOTNET_MSBUILD_SDK_RESOLVER_CLI_DIR = $env:DOTNET_ROOT
$env:DOTNET_MULTILEVEL_LOOKUP = '0'
$env:PATH = "$env:DOTNET_ROOT;$env:PATH"
```

Then standard `dotnet build`, `dotnet test`, etc. will work against individual projects.

### Running tests

**Unit tests** (no network required):

```powershell
# All unit tests
build.cmd -test

# Single test project via dotnet CLI (after setting env vars above)
cd src\System.ServiceModel.Primitives\tests
dotnet test

# Single test method
dotnet test --filter "FullyQualifiedName~MyTestClassName.MyTestMethod"
```

**Scenario/integration tests** ("outerloop" — require WCF test services running):

```powershell
# Runs self-hosted WCF test services automatically on Windows
build.cmd -test -testscope outerloop

# Against a remote WCF test service
build.cmd -test -testscope outerloop -- /p:ServiceUri=myServer
```

## Architecture

### NuGet Package ↔ Project Mapping

Each shipped NuGet package has a directory under `src/` with three subdirectories:

- **`src/`** — Implementation assembly (the code that ships in the NuGet package)
- **`ref/`** — Reference assembly (public API surface definition used at compile time)
- **`tests/`** — Unit tests for that package

The main packages and their dependency chain:

```
System.ServiceModel.Primitives  ← base (common types, channels, bindings, security)
├── System.ServiceModel.Http     ← HTTP bindings (BasicHttpBinding, WSHttpBinding, etc.)
├── System.ServiceModel.NetFramingBase ← common types for net.tcp and net.pipe framing
│   ├── System.ServiceModel.NetTcp    ← TCP binding (NetTcpBinding)
│   └── System.ServiceModel.NetNamedPipe ← Named pipe binding
├── System.ServiceModel.UnixDomainSocket ← Unix domain socket transport
└── System.ServiceModel.Federation  ← WS-Federation security
```

**Deprecated packages**: `System.ServiceModel.Duplex` and `System.ServiceModel.Security` are retired (type-forward to Primitives). Their assemblies are bundled inside the Primitives NuGet package for backward compatibility.

### System.Private.ServiceModel (Test Infrastructure)

This is **not** a shipped package. It contains all test infrastructure:

- **`tests/`** — All integration/scenario tests and shared test helpers:
  - `Common/Infrastructure/` — Custom xunit attributes (`[WcfFact]`, `[Condition]`, `[Issue]`, etc.)
  - `Common/Unit/` and `Common/Scenarios/` — Shared test helpers
  - `Scenarios/` — Integration tests organized by area (Binding/, Client/, Contract/, Security/, etc.)
- **`tools/`** — Test service applications and utilities:
  - `SelfHostedWcfService/` — .NET Framework self-hosted WCF test service
  - `SelfHostedCoreWcfService/` — CoreWCF-based self-hosted test service
  - `IISHostedWcfService/` — IIS-hosted test services; endpoint registration uses `[TestServiceDefinition]` attributes discovered via reflection in `TestDefinitionHelper`
  - `CertificateGenerator/` — Generates test certificates for security testing (revocation checks, expiration enforcement, chain trust validation, etc.)
  - `scripts/` — Scripts to start/stop test services and configure the test environment

### Solution Files

- **`System.ServiceModel.sln`** — Main solution (WCF client libraries + all tests)
- **`dotnet-svcutil.sln`** — The `dotnet-svcutil` CLI tool
- **`dotnet-svcutil.xmlserializer.sln`** — The `dotnet-svcutil-xmlserializer` tool

## Key Conventions

### Custom xunit Test Attributes

Tests use WCF-specific xunit attributes instead of standard `[Fact]`/`[Theory]`:

- **`[WcfFact]`** / **`[WcfTheory]`** — Required instead of `[Fact]`/`[Theory]`. Support conditional execution and issue tracking.
- **`[Condition(nameof(SomeCheck))]`** — Skips the test if the named static bool method/property returns false. Conditions can reference members on other classes via `"ClassName.MemberName"` syntax.
- **`[Issue(123)]`** — Marks a test as blocked by an open issue. Can be scoped by OS or framework: `[Issue(123, OS=OSID.Windows)]`.

These attributes live in `src/System.Private.ServiceModel/tests/Common/Infrastructure/xunit/`.

### Naming Conventions (from .editorconfig)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotnet/wcf](https://github.com/dotnet/wcf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
