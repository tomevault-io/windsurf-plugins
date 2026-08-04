---
trigger: always_on
description: Servidor MCP centralizado para armazenamento e busca de especificacoes tecnicas.
---

# SpecHub MCP Server

Servidor MCP centralizado para armazenamento e busca de especificacoes tecnicas.
Stack: Node.js + Mastra v1 + Sequelize + PostgreSQL/pgvector + @xenova/transformers (paraphrase-multilingual-MiniLM-L12-v2).

## Project layout

```
src/
  domain/                              # Enterprise business rules
    entities.ts                        # Spec, Task, ChangelogEntry interfaces
    repositories.ts                    # ISpecRepository, ITaskRepository, IChangelogRepository
    services.ts                        # IEmbeddingService interface
  application/                         # Use cases (orchestration)
    dto.ts                             # Input/Output DTOs
    use-cases/                         # One class per use case
  infrastructure/                      # Adapters, frameworks, drivers
    database/
      connection.ts                    # Sequelize instance (connectionString from DATABASE_URL)
      umzug.ts                         # Umzug instance (migration runner)
      migrations/                      # Individual migration files (001-xxx.ts, 002-xxx.ts, ...)
      models/                          # sequelize.define() model definitions
    repositories/                      # Sequelize implementations of domain repository interfaces
    services/                          # XenovaEmbeddingService (implements IEmbeddingService)
  container/                           # IoC / DI (Awilix)
    index.ts                           # buildContainer() — registers all singletons
    types.ts                           # AppContainer type alias
  mastra/                              # Interface / MCP presentation
    index.ts                           # Entry point: migrates, builds container, starts Mastra
    mcp.ts                             # createSpecHubMcpServer(container)
    tools/                             # Factory functions: createXxxTool(container)
tests/                                 # Vitest tests (mock use cases via Awilix asValue)
```

## Commands

```bash
npm run dev          # mastra dev (starts HTTP server on PORT, default 3456)
npm run build        # mastra build
npm run test         # vitest run
npm run typecheck    # tsc --noEmit
npm run lint         # full pipeline: typecheck → check:circular → eslint → test
npm run lint:eslint  # ESLint only (no-duplicate-imports, unused-vars, etc.)
npm run check:circular # madge: detect circular dependencies
docker compose up -d # start PostgreSQL/pgvector on :5434
```

## Architecture conventions

### Clean Architecture layers

- **Domain** (`src/domain/`): interfaces only. No imports from other layers. Defines entities, repository abstractions (ISP), and service abstractions.
- **Application** (`src/application/`): use cases with constructor injection. Depend only on domain interfaces. DTOs define input/output contracts.
- **Infrastructure** (`src/infrastructure/`): Sequelize models, repository implementations, embedding service. Implements domain interfaces.
- **Interface** (`src/mastra/`): MCP tools as factory functions receiving the container. Resolve use cases at runtime.
- **Container** (`src/container/`): Awilix IoC. All registrations are SINGLETON. Wires interfaces -> implementations.

### Dependency Injection (Awilix)

- `buildContainer()` in `src/container/index.ts` creates and registers all dependencies.
- Use cases receive repositories and services via **single object** constructor injection (PROXY mode).
- Awilix PROXY mode passes the entire cradle as one argument — constructors must receive `deps: Dependencies`.
- In tests, register mock use cases with `asValue(mockUseCase)`.

```ts
// Awilix PROXY mode: single deps object, not multiple params
class MyUseCase {
  private readonly specRepository: ISpecRepository

  constructor(deps: { specRepository: ISpecRepository }) {
    this.specRepository = deps.specRepository
  }
}

// Example: registering a mock in tests
import { createContainer, asValue } from 'awilix'
const mockUseCase = { execute: vi.fn() }
const container = createContainer()
container.register({ saveSpecUseCase: asValue(mockUseCase) })
```

### Tools (MCP)

- Each tool file exports a factory function that receives `AppContainer`.
- `inputSchema` defines the JSON contract. `outputSchema` defines the return shape.
- `execute(inputData)` resolves the use case from container and delegates.
- Tool IDs use `snake_case`: `save_spec`, `get_feature_overview`, `search_spec_context`.

### Database (Sequelize)

- `sequelize.define()` for model definitions in `src/infrastructure/database/models/`.
- Repositories implement domain interfaces, using Sequelize models internally.
- `raw: true` used for read queries; `create`/`update` for writes.
- Embedding serialization: `[0.1,0.2,...]` string stored in `embedding TEXT` column.
- UPSERT semantics: find by `(source_type, source_key)`, then INSERT or UPDATE.

### Migrations (Umzug)

Migrations are managed via **Umzug v3** with `SequelizeStorage`. Each migration is a separate file under `src/infrastructure/database/migrations/`.

**File naming:** `YYYYMMDDHHmmss-descriptive-name.ts` (e.g. `20260720180005-add-user-table.ts`)

**Template for new migrations:**

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cezarpretto/spec-hub](https://github.com/cezarpretto/spec-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
