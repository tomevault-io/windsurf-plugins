---
trigger: always_on
description: - Always use context7 when I need code generation, setup or configuration steps, or library/API documentation. This means you should automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.
---

- Always use context7 when I need code generation, setup or configuration steps, or library/API documentation. This means you should automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.
- Default admin account is username 'admin' and password 'changeme' . Sometime the password is 'changeme1'
- use serena MCP for code search and analysis without me explicitly having to ask. Prefer Serena's symbolic tools (`find_symbol`, `get_symbols_overview`, `find_referencing_symbols`, `search_for_pattern`) over grep/Grep for navigating code — they understand symbol relationships and are more precise.
- use superpowers for planning and workflow management without me having to ask
- IMPORTANT: Review docs/ARCHITECTURE_LESSONS.md before implementing node communication, state management, backup/restore, asynchronous operations, or database changes. These patterns prevent common mistakes.
- Only the backend talks to the Node. the Frontend never talks directly to the node.


## Git Worktree Policy

- **Core application changes** (anything affecting the running app): work directly on the live checkout at `/home/yeraze/Development/meshmonitor`
- **Documentation-only changes** (content for [https://meshmonitor.org](https://meshmonitor.org)) or **CI/CD pipeline-only changes**: may be done in a worktree
- When using a worktree, always start from the latest `origin/main` unless a specific branch or commit is required
- When working in a worktree, **deployment for user review is NOT required** before creating a PR — that requirement applies only to core application features on the main checkout

## Code Modification Rules

- After bulk find-and-replace or sed operations, always verify that modified functions have correct async/await signatures before running tests. Check that route handlers and callbacks are marked async if await was added inside them.

## Multi-Database Architecture (SQLite/PostgreSQL/MySQL)

MeshMonitor supports three database backends. When working with database code:

### Critical Rules
- **ALL database methods in DatabaseService must be async** - Use `methodNameAsync` naming convention
- **Use Drizzle ORM for queries** - Never write raw SQL that isn't database-agnostic
- **Test with SQLite first** - It's the default and most common deployment
- **Node IDs are BIGINT in PostgreSQL/MySQL** - Always coerce to Number when comparing (e.g., `Number(row.nodeNum)`)
- **Boolean columns differ by database** - SQLite uses 0/1, PostgreSQL uses true/false - Drizzle handles this
- **Schema definitions live in `src/db/schema/`** - One file per table, uses Drizzle's database-agnostic types

### Database Service Architecture
```
src/services/database.ts      # Main service - facade over repositories
src/db/
  schema/                     # Drizzle schema definitions (database-agnostic)
  repositories/               # Domain-specific async repositories
  drivers/
    sqlite.ts                 # SQLite driver (better-sqlite3)
    postgres.ts               # PostgreSQL driver (pg)
    mysql.ts                  # MySQL driver (mysql2)
```

### Adding New Database Methods
1. Add async method to the appropriate repository in `src/db/repositories/`
2. Expose it through DatabaseService with `Async` suffix
3. Use Drizzle query builders - they generate correct SQL for each database
4. For raw SQL, use `db.drizzleDbType` to check database type and adjust syntax
5. **IMPORTANT**: When adding routes that use database methods, ensure tests mock the async versions

### Test Mocking Pattern
When tests mock DatabaseService, they must provide async method mocks for authMiddleware:
```typescript
(DatabaseService as any).findUserByIdAsync = vi.fn().mockResolvedValue(user);
(DatabaseService as any).findUserByUsernameAsync = vi.fn().mockResolvedValue(null);
(DatabaseService as any).checkPermissionAsync = vi.fn().mockResolvedValue(true);
(DatabaseService as any).getUserPermissionSetAsync = vi.fn().mockResolvedValue({ resources: {}, isAdmin: false });
```

### Database Detection
- `DATABASE_URL` env var triggers PostgreSQL or MySQL based on protocol
- No `DATABASE_URL` = SQLite (default)
- Check `databaseService.drizzleDbType` for runtime database type ('sqlite' | 'postgres' | 'mysql')
- When sending messages for testing, use the "gauntlet" channel. Never send on Primary!
- Always start the Dev environment via docker, and make sure to 'build' first
- You can't have both the Docker and the local npm version running at the same time, or they interfere. If you want to switch, you need to let me know.
- Load up the system on port 8080
- Never push directly to main, always push to a branch.
- Our container doesn't have sqlite3 as a binary available.
- When testing locally, use the docker-compose.dev.yml to build the local code.  Also, always make sure the proper code was deployed once the container is launched.
- Official meshtastic protobuf definitions can be found at https://github.com/meshtastic/protobufs/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yeraze/meshmonitor](https://github.com/Yeraze/meshmonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
