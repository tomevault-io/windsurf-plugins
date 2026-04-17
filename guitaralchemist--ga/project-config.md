---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build & Run
```powershell
# Full solution build (.NET 10, all targets)
dotnet build AllProjects.slnx -c Debug

# Start all services via Aspire (MongoDB, GaApi, Chatbot, dashboards)
pwsh Scripts/start-all.ps1 -Dashboard

# Daily dev restart without rebuild
pwsh Scripts/start-all.ps1 -NoBuild -Dashboard

# One-time setup (install prereqs, restore NuGet/npm, build)
pwsh Scripts/setup-dev-environment.ps1
```

### Testing
```powershell
# Full test suite
dotnet test AllProjects.slnx
pwsh Scripts/run-all-tests.ps1

# Backend only (faster)
pwsh Scripts/run-all-tests.ps1 -BackendOnly -SkipBuild

# Playwright E2E only
pwsh Scripts/run-all-tests.ps1 -PlaywrightOnly

# Single test class
dotnet test --filter "FullyQualifiedName~GrothendieckServiceTests"

# Single test method
dotnet test --filter "FullyQualifiedName~GrothendieckServiceTests.ComputeICV.ShouldComputeICV_ForCMajorScale"
```

### Code Quality
```powershell
# Format (run before committing; pre-commit hook enforces this)
dotnet format AllProjects.slnx

# Verify format without changing files (used in CI)
dotnet format AllProjects.slnx --verify-no-changes

# Install pre-commit hook (enforces format + build on each commit)
pwsh Scripts/install-git-hooks.ps1
```

### Frontend
```bash
# Sandbox development server (in ReactComponents/ga-react-components)
npm ci && npm run dev

# Production build
npm run build

# Lint
npm run lint
```

### Verification (required before claiming success)
```powershell
dotnet build AllProjects.slnx -c Debug   # Build passes
dotnet test AllProjects.slnx             # Tests pass
```
```bash
npm run build   # Frontend builds (in ReactComponents/ga-react-components)
npm run lint    # Frontend lint clean
```

### Monitoring
- Aspire dashboard: `https://localhost:15001`
- Jaeger tracing: `http://localhost:16686`
- Mongo Express: `http://localhost:8081`
- Health endpoint: `https://localhost:7001/health`

## Architecture

### Five-Layer Dependency Model

The codebase enforces a strict bottom-up dependency graph. Each layer may only depend on layers below it.

| Layer | Project(s) | Purpose |
|---|---|---|
| **1 – Core** | `GA.Core`, `GA.Domain.Core` | Pure domain primitives: Note, Interval, PitchClass, Fretboard types |
| **2 – Domain** | `GA.Business.Core`, `GA.Business.Config`, `GA.BSP.Core` | Business logic, YAML configuration, BSP geometry |
| **3 – Analysis** | `GA.Business.Core.Harmony`, `GA.Business.Core.Fretboard` | Chord/scale analysis, voice leading, spectral/topological analysis |
| **4 – AI/ML** | `GA.Business.ML` | Semantic indexing, Ollama/ONNX embeddings, vector search, Spectral RAG, tab solving |
| **5 – Orchestration** | `GA.Business.Core.Orchestration`, `GA.Business.Assets`, `GA.Business.Intelligence` | High-level workflows, IntelligentBSPGenerator, curation |

**Rule**: AI code belongs in `GA.Business.ML`. Orchestration code (e.g., `IntelligentBSPGenerator`) belongs in `GA.Business.Core.Orchestration`, not in low-level libraries.

### Applications (`Apps/`)

| App | Tech | Role |
|---|---|---|
| `ga-server/GaApi` | ASP.NET Core, SignalR, HotChocolate (GraphQL) | Main REST + GraphQL API, chatbot hub, vector search |
| `GaChatbot` | .NET | Guitar Alchemist Chatbot (Ollama-powered RAG) |
| `ga-client` | React 18, Vite, MUI, React Three Fiber | React frontend with 3D fretboard visualization |
| `GaMusicTheoryLsp` | F# | Language Server Protocol for the music theory DSL |
| `FloorManager` | Blazor | BSP room viewer |
| `AllProjects.AppHost` | .NET Aspire | Orchestrates all services for local dev |

### Key Cross-Cutting Modules

- **`GA.Business.DSL`** (F#): Music theory DSL – parsers (VexTab, GuitarPro, MIDI, MusicXML, AsciiTab), generators, LSP. Uses FParsec.
- **`GA.Business.Config`** (F#/YAML): Static domain data – scales, instruments, voicings, tunings. Loaded via YamlDotNet with PascalCase naming convention.
- **`GaMcpServer`** / **`mcp-servers/`**: MCP server tools. Update manifests here when adding new agent integrations.
- **`ReactComponents/ga-react-components`**: Shared React component library; production bundle consumed by `Apps/ga-client`.

### OPTIC-K Embedding Schema (v1.6)
216-dimension musical embedding vector used throughout the RAG pipeline:
- **STRUCTURE** (dims 6–29, weight 0.45): Pitch-class invariants – primary driver of musical similarity.
- **MORPHOLOGY** (dims 30–53, weight 0.25): Physical fretboard geometry.
- **CONTEXT** (dims 54–65, weight 0.20): Harmonic functionality and voice motion.
- **SYMBOLIC** (dims 66–77, weight 0.10): Manual tags.
- Canonical implementation: `Common/GA.Business.ML/Embeddings/EmbeddingSchema.cs`.
- Never change the dimension count without a coordinated full-system re-index.

### Vector Search Stack
- **ONNX Runtime** + `all-MiniLM-L6-v2` (384-dim text embeddings), or OPTIC-K (216-dim musical embeddings).
- Backends: `InMemoryVectorIndex`, `QdrantVectorIndex`, `MongoDbVectorSearchStrategy`, `ILGPUVectorSearchStrategy` (CUDA).
- Document IDs must be deterministic: `entity_type_discriminator` (never random UUIDs).

## C# Standards (C# 14 / .NET 10)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuitarAlchemist) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
