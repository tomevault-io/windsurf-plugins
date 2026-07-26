---
trigger: always_on
description: This document provides essential guidelines for AI agents (like GitHub Copilot) working on the **SlimFaas** and **SlimData** projects. It covers compilation strategies, execution commands, testing procedures, and documentation requirements.
---

# Agent Guidelines for SlimFaas Development

## 🎯 Overview

This document provides essential guidelines for AI agents (like GitHub Copilot) working on the **SlimFaas** and **SlimData** projects. It covers compilation strategies, execution commands, testing procedures, and documentation requirements.

---

## 📦 Core Technologies

### SlimFaas & SlimData: AOT Compilation

**Both SlimFaas and SlimData are compiled using Ahead-of-Time (AOT) compilation**, a .NET feature that compiles IL code directly to native machine code at build time.

#### Why AOT?

- **Slim Footprint**: Reduced binary size and memory usage
- **Faster Startup**: No JIT compilation overhead
- **Production Ready**: Ideal for containerized FaaS workloads
- **Cold-start Optimization**: Functions wake up instantly from scale-to-zero

#### AOT Configuration

Both projects have `<PublishAot>true</PublishAot>` in their `.csproj` files:

- **SlimFaas** (`src/SlimFaas/SlimFaas.csproj`):
  - Target Framework: `.NET 10.0`
  - Full trimming enabled
  - Symbols stripped
  - Unsafe blocks allowed for performance

- **SlimData** (`src/SlimData/SlimData.csproj`):
  - Target Framework: `.NET 10.0`
  - Optimization preference: Size
  - Full trimming enabled

#### Important AOT Considerations

- **Reflection Limitations**: Minimize runtime reflection; use code generation where needed
- **Type Safety**: Ensure all types used in serialization are discoverable at compile time
- **Native Dependencies**: Be careful with P/Invoke calls; verify they work across platforms
- **Dependencies**: Only use NuGet packages with AOT support (e.g., `KubernetesClient.Aot`, `MemoryPack`, `prometheus-net`)

---

## 🚀 Running the Project

### Prerequisites

- **.NET SDK**: Version `10.0.103` or later (see `global.json`)
- **Node.js**: Version `24` or later (for UI/dashboard builds)
- **Docker** or **Podman** (optional, for containerized deployments)

### Building

```bash
# Build the entire solution
dotnet build

# Build with AOT compilation (creates native executable)
dotnet publish -c Release

# Build a specific project
dotnet build src/SlimFaas/SlimFaas.csproj
```

### Running Locally

```bash
# Run SlimFaas in development mode
dotnet run --project src/SlimFaas/SlimFaas.csproj

# Run with Kubernetes integration (requires k8s cluster)
dotnet run --project src/SlimFaas/SlimFaas.csproj

# Run examples
dotnet run --project src/Fibonacci/Fibonacci.csproj
dotnet run --project demo/
```

### Docker

```bash
# Build Docker image
docker build -t slimfaas:latest .

```

---

## 🧪 Unit Tests

SlimFaas maintains comprehensive test coverage across multiple test projects.

### Test Projects

- **SlimFaas.Tests** (`tests/SlimFaas.Tests/`)
  - Core SlimFaas functionality: HTTP proxy, workers, scaling logic
  - Metrics, replicas synchronization, history
  - Data handling and endpoints

- **SlimData.Tests** (`tests/SlimData.Tests/`)
  - Raft-based cluster consensus
  - Key-value operations
  - File-based persistence
  - Command serialization

- **SlimFaasMcp.Tests** (`tests/SlimFaasMcp.Tests/`)
  - MCP (Model Context Protocol) integration tests

- **SlimFaasKafka.Tests** (`tests/SlimFaasKafka.Tests/`)
  - Kafka connector and lag monitoring

### Running Tests

```bash
# Run all unit tests
dotnet test

# Run with code coverage
dotnet test --collect "Code Coverage;Format=cobertura"

# Run specific test project
dotnet test tests/SlimFaas.Tests/SlimFaas.Tests.csproj

# Run specific test with verbose output
dotnet test --filter "ClassName=YourTestClass" --verbosity detailed

# Watch mode (re-run on file changes)
dotnet watch test
```

### Code Coverage

Code coverage reports are generated during CI/CD and stored in `TestResults/` directories:

```bash
# View coverage (after test run)
open coveragereport/index.html
```

### Testing Best Practices

✅ **Always**:
- Write tests for new features or bug fixes
- Use meaningful test names (e.g., `WhenScalingUpWith_TenRequests_ShouldCreateNewReplicas()`)
- Mock external dependencies (Kubernetes API, HTTP calls)
- Test both success and failure paths
- Ensure tests are AOT-compatible (avoid reflection where possible)

❌ **Never**:
- Leave failing tests in the codebase
- Ignore test failures in CI/CD
- Write tests that depend on external services
- Use hardcoded delays instead of proper async/await patterns

---

## 📚 Documentation Requirements

### Golden Rule: Always Update Documentation

**Every code change that affects user-facing behavior, configuration, or architecture MUST be accompanied by documentation updates.**

### Documentation Files to Update

#### 1. **README.md** (Root)
Located at `/README.md`, this is the first impression:
- Update project description if scope changes
- Keep feature list current with new capabilities
- Update performance benchmarks if AOT compilation improves metrics
- Add/remove links to documentation sections as needed

#### 2. **Documentation Folder** (`./documentation/`)
These files are **automatically published to the website** (https://slimfaas.dev):

- **`get-started.md`** – Deployment instructions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SlimPlanet/SlimFaas](https://github.com/SlimPlanet/SlimFaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
