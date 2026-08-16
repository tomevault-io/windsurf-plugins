---
trigger: always_on
description: This document provides essential guidelines for AI agents (like GitHub Copilot) working on the **SlimFaas**, **SlimData**, **SlimFaasMcp**, and client packages. It covers compilation strategies, execution commands, testing procedures, and documentation requirements.
---

# Agent Guidelines for SlimFaas Development

## 🎯 Overview

This document provides essential guidelines for AI agents (like GitHub Copilot) working on the **SlimFaas**, **SlimData**, **SlimFaasMcp**, and client packages. It covers compilation strategies, execution commands, testing procedures, and documentation requirements.

---

## 📦 Core Technologies

### SlimFaas, SlimData & SlimFaasMcp: AOT Compilation

**SlimFaas, SlimData, and SlimFaasMcp are compiled using Ahead-of-Time (AOT) compilation**, a .NET feature that compiles IL code directly to native machine code at build time.

#### Why AOT?

- **Slim Footprint**: Reduced binary size and memory usage
- **Faster Startup**: No JIT compilation overhead
- **Production Ready**: Ideal for containerized FaaS workloads
- **Cold-start Optimization**: Functions wake up instantly from scale-to-zero

#### AOT Configuration

The native .NET services below have `<PublishAot>true</PublishAot>` in their `.csproj` files:

- **SlimFaas** (`src/SlimFaas/SlimFaas.csproj`):
  - Target Framework: `.NET 10.0`
  - Full trimming enabled
  - Symbols stripped
  - Unsafe blocks allowed for performance

- **SlimData** (`src/SlimData/SlimData.csproj`):
  - Target Framework: `.NET 10.0`
  - Optimization preference: Size
  - Full trimming enabled

- **SlimFaasMcp** (`src/SlimFaasMcp/SlimFaasMcp.csproj`):
  - Target Framework: `.NET 10.0`
  - Full trimming enabled
  - Symbols stripped
  - Builds `ClientApp` with `npm ci` and `npm run build` before build/publish

#### Important AOT Considerations

- **Reflection Limitations**: Minimize runtime reflection; use code generation where needed
- **Type Safety**: Ensure all types used in serialization are discoverable at compile time
- **Native Dependencies**: Be careful with P/Invoke calls; verify they work across platforms
- **Dependencies**: Only use NuGet packages with AOT support (e.g., `KubernetesClient.Aot`, `MemoryPack`, `prometheus-net`)
- **Source-generated serialization**: Add new JSON payloads to existing `JsonSerializerContext` partials (for example `src/SlimFaasMcp/AppJsonContext.cs` or `src/SlimFaas/Local/ProcessControlContracts.cs`) and keep SlimData command payloads `MemoryPackable`.

---

## 🚀 Running the Project

### Prerequisites

- **.NET SDK**: Version `10.0.103` or later (see `global.json`)
- **Node.js**: Version `24` or later (for UI/dashboard builds)
- **pnpm**: Version `10.14.0` is used by CI for `src/SlimFaasSite` and `src/SlimFaasPlanetSaver`
- **Python/UV**: Python `>=3.10` with `uv` for `client/python/slimfaas-client`
- **Docker** or **Podman** (optional, for containerized deployments)

### Building

```bash
# Build the entire solution
dotnet build

# Fast backend-only SlimFaas build (skip embedded dashboard ClientApp)
dotnet build src/SlimFaas/SlimFaas.csproj -p:SkipClientAppBuild=true

# Build with AOT compilation (creates native executable)
dotnet publish -c Release

# Build a specific project
dotnet build src/SlimFaas/SlimFaas.csproj

# Build the documentation site
(cd src/SlimFaasSite && pnpm install --frozen-lockfile && pnpm build)
```

### Running Locally

```bash
# Run SlimFaas in development mode
dotnet run --project src/SlimFaas/SlimFaas.csproj

# Run with Kubernetes integration (requires k8s cluster)
dotnet run --project src/SlimFaas/SlimFaas.csproj

# Run examples
dotnet run --project src/Fibonacci/Fibonacci.csproj
dotnet run --project src/FibonacciBatch/FibonacciBatch.csproj

# Validate and run the native local demo (paths are relative to the src/SlimFaas launch profile)
dotnet run --project src/SlimFaas -- local validate -f ../../slimfaas.local.yaml
dotnet run --project src/SlimFaas -- local up -f ../../slimfaas.local.yaml
```

Use repeatable `-f` overlays such as `slimfaas.local.dev.yaml` or `slimfaas.local.debug.yaml`; `debugUrl` routes a function to an IDE process in native local mode.

### Docker

```bash
# Build Docker image
docker build -t slimfaas:latest .

# Run local Compose demo
docker-compose up

# Podman Compose on macOS
chmod +x ./run-podman-compose.sh
./run-podman-compose.sh up -d
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

- **SlimFaasClient.Tests** (`client/dotnet/SlimFaasClient/tests/`)
  - .NET WebSocket client registration, async handling, sync streaming

- **Python slimfaas-client tests** (`client/python/slimfaas-client/tests/`)
  - Python WebSocket client behavior with `uv run pytest`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SlimPlanet/SlimFaas](https://github.com/SlimPlanet/SlimFaas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
