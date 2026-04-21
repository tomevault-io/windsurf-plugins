---
trigger: always_on
description: **Generated:** 2026-02-01
---

# OVERSEER PROJECT KNOWLEDGE BASE

**Generated:** 2026-02-01  
**Commit:** 829fb09  
**JJ Change:** mptvnovo

**Overseer** (`os`) - Codemode MCP server for agent task management. SQLite-backed, native VCS (jj-lib + gix). JJ-first.

## ARCHITECTURE

```
+-------------------------------------------------------------+
|                     Overseer (Node MCP)                     |
|  - Single "execute" tool (codemode pattern)                 |
|  - VM sandbox with tasks/learnings APIs                     |
|  - Spawns CLI, parses JSON                                  |
+-------------------------------------------------------------+
                              |
                              v
+-------------------------------------------------------------+
|                      os (Rust CLI)                          |
|  - All business logic                                       |
|  - SQLite storage                                           |
|  - Native VCS: jj-lib (jj) + gix (git)                      |
|  - JSON output mode for MCP                                 |
+-------------------------------------------------------------+
```

## STRUCTURE

```
overseer/
├── overseer/                # Rust CLI package (binary: os)
│   └── src/
│       ├── main.rs          # Entry (clap CLI)
│       ├── commands/        # Subcommand handlers
│       ├── core/            # TaskService, WorkflowService, context
│       ├── db/              # SQLite repos
│       └── vcs/             # jj-lib + gix backends
│
├── mcp/                     # Node MCP wrapper
│   └── src/
│       ├── index.ts         # Entry (stdio transport)
│       ├── server.ts        # execute tool registration
│       ├── executor.ts      # VM sandbox, CLI bridge
│       └── api/             # tasks/learnings APIs
│
├── npm/                     # Publishing (platform-specific binaries)
│   ├── overseer/            # Main package (routing wrapper)
│   └── scripts/             # Platform package generation
│
├── skills/                  # Agent skills (skills.sh compatible)
│   ├── overseer/            # Task management skill
│   └── overseer-plan/       # Plan-to-task conversion skill
│
├── ui/                      # Task Viewer webapp (Hono + Vite + React)
│   └── src/
│       ├── api/             # Hono API server
│       ├── client/          # React SPA
│       └── types.ts         # Shared types
│
└── docs/                    # Reference documentation
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add CLI command | `overseer/src/commands/` | Add in mod.rs, wire in main.rs |
| Add MCP API | `mcp/src/api/` | Export in api/index.ts |
| Task CRUD | `overseer/src/db/task_repo.rs` | SQL layer |
| Task business logic | `overseer/src/core/task_service.rs` | Validation, hierarchy (1407 lines) |
| Task workflow (start/complete) | `overseer/src/core/workflow_service.rs` | VCS integration (816 lines) |
| VCS operations | `overseer/src/vcs/` | jj.rs (primary), git.rs (fallback) |
| Error types | `overseer/src/error.rs` | OsError enum |
| Types/IDs | `overseer/src/types.rs`, `overseer/src/id.rs` | Domain types, ULID |
| UI API routes | `ui/src/api/routes/` | Hono route handlers |
| UI components | `ui/src/client/components/` | React components |
| UI queries | `ui/src/client/lib/queries.ts` | TanStack Query hooks |
| UI theme | `ui/src/client/styles/global.css` | Tailwind v4 CSS tokens |

## KEY DECISIONS

| Decision | Choice | Why |
|----------|--------|-----|
| CLI binary | `os` | Short, memorable |
| Storage | SQLite | Concurrent access, queries |
| VCS primary | jj-lib | Native perf, no spawn |
| VCS fallback | gix | Pure Rust, no C deps |
| VCS semantics | Unified stacking | Both jj & git behave identically |
| IDs | ULID | Sortable, coordination-free |
| Task hierarchy | 3 levels max | Milestone(0) -> Task(1) -> Subtask(2) |
| Error pattern | `thiserror` | Ergonomic error handling |

## TYPE SYNC (Rust <-> TS)

Types must stay in sync between `overseer/src/types.rs`, `overseer/src/core/context.rs`, and `mcp/src/types.ts`:
- `TaskId`: Newtype (Rust) / Branded type (TS), `task_` prefix + 26-char ULID
- `LearningId`: Newtype / Branded, `lrn_` prefix
- `Task`, `Learning`, `TaskContext`: Identical shapes
- `InheritedLearnings`: Rust struct in `context.rs` has `own`, `parent`, `milestone`; TS `InheritedLearnings` in `mcp/src/types.ts` matches
- Rust uses `serde(rename_all = "camelCase")` -> JSON matches TS interfaces

**Note:** The `InheritedLearnings` type in `overseer/src/types.rs` (with only `milestone` and `parent`) is for import/export schema compatibility. The actual runtime type used for `TaskWithContext` is in `context.rs` and includes `own`.

**When changing constrained types (e.g., Priority range):**
1. Rust: `types.rs`, validation in `task_service.rs`, CLI args in `commands/task.rs`
2. TypeScript types: `generated/types.ts`, `host/src/types.ts`, `ui/src/types.ts`
3. TypeScript decoders: `host/src/decoder.ts`, `ui/src/decoder.ts`
4. TypeScript API interfaces: `host/src/api/tasks.ts`, `host/src/ui.ts`
5. UI input constraints: Any `min/max` on number inputs

## CONVENTIONS

- **Result everywhere**: All fallible ops return `Result<T, E>`
- **TaggedError (TS)**: Errors use `_tag` discriminator
- **No `any`**: Strict TypeScript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmmulroy/overseer](https://github.com/dmmulroy/overseer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
