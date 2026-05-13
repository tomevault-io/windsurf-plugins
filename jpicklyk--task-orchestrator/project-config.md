---
trigger: always_on
description: A Kotlin-based MCP server providing hierarchical work item management with dependency tracking, note schemas, and role-based workflow automation.
---

## Project: MCP Task Orchestrator

A Kotlin-based MCP server providing hierarchical work item management with dependency tracking, note schemas, and role-based workflow automation.

**Key Technologies:**
- Kotlin 2.2.0 with Coroutines
- Exposed ORM 1.0.0-beta-2 for SQLite
- MCP SDK 0.9.0 (with Ktor Streamable HTTP transport)
- Flyway for database migrations
- Gradle with Kotlin DSL / Docker

## Build Commands

```bash
./gradlew build                        # fat JAR → current/build/libs/
./gradlew clean build
./gradlew test
./gradlew test --tests "*ToolTest"

java -jar current/build/libs/mcp-task-orchestrator-*.jar

# Docker (most common)
docker build -t task-orchestrator:dev .
docker run --rm -i \
  -v mcp-task-data:/app/data \
  -v "$(pwd)"/.taskorchestrator:/project/.taskorchestrator:ro \
  -e AGENT_CONFIG_DIR=/project \
  task-orchestrator:dev
```

## Architecture

Source lives under `current/`.

**Package root:** `io.github.jpicklyk.mcptask.current`
**Source root:** `current/src/main/kotlin/io/github/jpicklyk/mcptask/current/`

```
domain/
  model/       — WorkItem, Note, Dependency, Role, Priority, RoleTransition, LifecycleMode, WorkItemSchema
  repository/  — WorkItemRepository, NoteRepository, DependencyRepository, RoleTransitionRepository

application/
  tools/items/      — ManageItemsTool, QueryItemsTool
  tools/notes/      — ManageNotesTool, QueryNotesTool
  tools/dependency/ — ManageDependenciesTool, QueryDependenciesTool
  tools/workflow/   — AdvanceItemTool, GetNextStatusTool, GetNextItemTool, GetBlockedItemsTool, GetContextTool
  tools/compound/   — CreateWorkTreeTool, CompleteTreeTool
  service/          — RoleTransitionHandler, NoteSchemaService, CascadeDetector, WorkTreeExecutor

infrastructure/
  database/schema/      — WorkItemsTable, NotesTable, DependenciesTable, RoleTransitionsTable
  database/schema/management/ — DirectDatabaseSchemaManager, FlywayDatabaseSchemaManager, SchemaManagerFactory
  repository/           — SQLite implementations, RepositoryProvider
  config/               — YamlWorkItemSchemaService (typealias YamlNoteSchemaService)

interfaces/mcp/
  CurrentMcpServer.kt, McpToolAdapter.kt
```

**Entry point:** `current/src/main/kotlin/io/github/jpicklyk/mcptask/current/CurrentMain.kt`

## Modes of Operation

- **Orchestration** (default) — orchestrator pushes items through phases via `advance_item`
- **Claim** (opt-in) — consumers pull work via `claim_item`, holding TTL-based ownership before advancing

The optional `actor_authentication` config block adds JWKS-based identity verification — independent of claim mode (claim works without it). See [`current/docs/fleet-deployment.md`](current/docs/fleet-deployment.md).

## Trait System (Orchestration Signals)

Traits are **composable orchestration signals** declared in `.taskorchestrator/config.yaml` under the `traits:` key. They are NOT merely note requirements. Each trait carries three dimensions:

1. **Note requirements** -- notes with `key`, `role`, `required` that merge into an item's resolved schema and enforce gates
2. **Guidance** -- `guidance` text on each note telling agents HOW to fill it (context, constraints, structure)
3. **Skill routing** -- optional `skill` pointer (e.g., `skill: "migration-review"`) that routes evaluation to a specialized skill

**Resolution flow:** `ToolExecutionContext.resolveSchema(item)` merges trait notes from two sources:
- `defaultTraits` on the schema type definition (always applied to items of that type)
- Per-item `traits` from the item's `properties` JSON bag (applied via `PropertiesHelper.extractTraits()`)

Base schema note keys win on duplicates; first-trait-in-order wins for duplicate trait keys.

**Example:** An item typed `feature-task` with trait `needs-migration-review` gets the base `feature-task` notes PLUS the `migration-assessment` note (queue phase, required, with `migration-review` skill pointer and guidance about SQLite table recreation patterns). The orchestrator sees this merged schema via `get_context(itemId=...)` and routes accordingly -- dispatching a migration-specialized agent or invoking the migration-review skill.

**Key files:**

| What | Path |
|------|------|
| Trait definitions | `.taskorchestrator/config.yaml` -> `traits:` section |
| Schema resolution + trait merging | `current/.../application/tools/ToolExecutionContext.kt` -> `resolveSchema()`, `mergeTraits()` |
| Properties helper | `current/.../application/tools/PropertiesHelper.kt` -> `extractTraits()`, `mergeTraits()` |
| Domain models | `WorkItemSchema.kt` (`defaultTraits`), `NoteSchemaEntry.kt` (`skill`, `guidance`) |

## Tight Coupling Areas

### ToolExecutionContext
Constructed in `CurrentMcpServer.kt` as `ToolExecutionContext(repositoryProvider, noteSchemaService)`. Adding a new service dependency requires updating **both** the context class and the server construction site.

### DirectDatabaseSchemaManager
Maintains a manually-ordered table list in foreign-key dependency order. New tables must be inserted at the correct position — the compiler cannot detect wrong ordering.

## Configuration Directory (AGENT_CONFIG_DIR)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpicklyk/task-orchestrator](https://github.com/jpicklyk/task-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
