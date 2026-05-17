---
trigger: always_on
description: **OpenDeepWiki** is an AI-driven code knowledge base system built on **.NET 9** and **Semantic Kernel**. It analyzes code repositories, generates documentation, creates directory structures, and supports MCP (Model Context Protocol) for AI integration.
---

# OpenDeepWiki Development Guide for AI Agents

## Project Overview

**OpenDeepWiki** is an AI-driven code knowledge base system built on **.NET 9** and **Semantic Kernel**. It analyzes code repositories, generates documentation, creates directory structures, and supports MCP (Model Context Protocol) for AI integration.

### Core Purpose
- Convert GitHub/GitLab/Gitee repositories into searchable knowledge bases
- Auto-generate documentation, READMEs, and code analysis via LLM
- Support multiple AI providers (OpenAI, AzureOpenAI, Anthropic)
- Provide MCP endpoints for AI agents to query repository knowledge

---

## Architecture

### Full-Stack Structure
```
Backend: .NET 9 ASP.NET Core + Entity Framework Core + Semantic Kernel
Frontend: React 19 + TypeScript + Vite + TailwindCSS + Shadcn/ui
Database: SQLite/PostgreSQL/MySQL/SQL Server (configurable)
Deployment: Docker Compose or Sealos
```

### Backend Layer Breakdown

**`src/KoalaWiki/`** - Main ASP.NET Core application
- **`BackendService/`** - Background task orchestration (warehouse sync, document processing)
- **`KoalaWarehouse/`** - Core document analysis engine:
  - **`Pipeline/`** - Resilient document processing pipeline with 5 ordered steps
  - **`GenerateThinkCatalogue/`** - AI-powered directory structure generation
  - **`DocumentPending/`** - Incomplete document task handling
  - **`MiniMapService.cs`** - Generates knowledge graphs via Mermaid

**`KoalaWiki.Core/`** - Data access layer
- **`DataAccess/IKoalaWikiContext.cs`** - DbSet definitions for 18+ entity types
- **`ServiceExtensions.cs`** - DI registration for database providers

**`KoalaWiki.Domains/`** - Domain models
- **`Warehouse.cs`** - Repository metadata and configuration
- **`Document.cs`** - Document content and metadata
- **`DocumentFile/`** - File structure and catalog definitions
- **`FineTuning/`** - Training dataset generation
- **`MCP/`** - Model Context Protocol entities

**`Provider/`** - Database implementations
- `KoalaWiki.Provider.PostgreSQL`
- `KoalaWiki.Provider.MySQL`
- `KoalaWiki.Provider.SqlServer`

### Frontend Layer Breakdown

**`web-site/src/`** - React application
- **`pages/`** - Route-based page components: `home`, `auth`, `admin`, `repository`, `chat`
- **`components/`** - Reusable UI components (RepositoryLayout, AdminLayout)
- **`services/`** - HTTP API clients and API wrappers
- **`stores/`** - Zustand state management stores
- **`i18n/`** - Internationalization (Chinese, English, French)
- **`routes/`** - React Router configuration with lazy loading

---

## Critical Data Flows

### 1. Repository Analysis Flow (README from README.md)
```
Clone Repository → .gitignore Filtering → Directory Scanning → 
AI Smart Filter (if file count > threshold) → Directory JSON → 
Generate README → Project Classification → Project Overview → 
Save to Database → Generate Task List (Think Catalogue) → 
Process Documents Recursively → Generate Commit Log
```

### 2. Document Processing Pipeline (5-Step Architecture)
Located in `KoalaWarehouse/Extensions/ServiceCollectionExtensions.cs`:

**Execution Order:**
1. **ReadmeGenerationStep** - Generate README.md
2. **CatalogueGenerationStep** - Create directory structure
3. **ProjectClassificationStep** - Classify project type
4. **DocumentStructureGenerationStep** - Build document TOC
5. **DocumentContentGenerationStep** - Generate document content

**Key Classes:**
- `ResilientDocumentProcessingPipeline` - Wraps pipeline with retry/fallback logic
- `DocumentProcessingContext` - Carries data through steps
- `DocumentProcessingOrchestrator` - Orchestrates with OpenTelemetry tracing

### 3. AI Kernel Initialization (KernelFactory Pattern)
`KernelFactory.GetKernel()` initializes Semantic Kernel with:
- **LLM Provider Selection**: OpenAI or AzureOpenAI via `OpenAIOptions.ModelProvider`
- **Plugins Loaded**:
  - Code Analysis plugins (in `plugins/CodeAnalysis/`) with `.skprompt.txt` prompts
  - FileTool plugin - reads repository files with token limits
  - AgentTool plugin - MCP integration
  - Dynamic MCP service loading from `DocumentOptions.McpStreamable`
- **Custom HttpClient** - Handles gzip/brotli decompression

---

## Key Development Workflows

### Build & Run

**Frontend:**
```bash
cd web-site
npm install
npm run dev          # Dev server at localhost:5173
npm run build        # Build to ../src/KoalaWiki/wwwroot
npm run build:analyze  # Bundle analysis
npm run lint         # ESLint check
```

**Backend:**
```bash
dotnet build KoalaWiki.sln
dotnet run --project src/KoalaWiki/KoalaWiki.csproj
# API at http://localhost:5085, OpenAPI at /scalar
```

**Docker (with make/Makefile):**
```bash
make build              # Build all images
make build-frontend     # Frontend only
make dev               # Run all services with logs
make dev-backend       # Backend only
make build-arm         # ARM64 architecture
make build-amd         # AMD64 architecture
```

### Database Migrations

Entity Framework Core migrations (in `KoalaWiki.Core/`):
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AIDotNet/OpenDeepWiki](https://github.com/AIDotNet/OpenDeepWiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
