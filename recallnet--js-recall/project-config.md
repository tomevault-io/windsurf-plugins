---
trigger: always_on
description: - All database schemas must be placed in `packages/db/src/schemas/` with appropriate subdirectories (`core/`, `auth/`, etc.)
---

# API Engineering Standards (apps/api)

## Database & Schema Management

### Database Schema Organization
- All database schemas must be placed in `packages/db/src/schemas/` with appropriate subdirectories (`core/`, `auth/`, etc.)
- Schema files must export table definitions and be properly imported in `core/defs.ts`
- Always generate migrations using `pnpm db:gen-migrations` after schema changes which will automatically create the sql migration files
- Repository classes must be placed in `apps/api/src/database/repositories/` and follow the naming pattern `*-repository.ts`
- Use the established repository pattern with dependency injection into manager services
- Include proper indexes for foreign keys and frequently queried columns

### Database Computation Patterns
- **SQL-First Philosophy**: Always prefer database aggregations over in-memory processing
  - Use SQL `SUM()`, `AVG()`, `COUNT()` instead of fetching all rows and calculating in JavaScript
  - Use `DISTINCT ON` with proper indexes for latest-record queries
  - Use database-level `GROUP BY` for grouping operations
  - Use CTEs (Common Table Expressions) for complex multi-step queries
  - Use `ORDER BY` in SQL instead of JavaScript `.sort()`
  - For complex sorts (like `DISTINCT ON` + different `ORDER BY`), use subqueries with `.as()` in Drizzle
  - Example: Getting latest summaries should use `DISTINCT ON (agent_id)` not fetch all and filter in memory
- **Response Size Management**:
  - Never return unbounded result sets for in-memory processing
  - Always aggregate at database level when only totals are needed
  - Example: Use `SUM(total_volume)` in SQL vs fetching 1000+ records to sum in JavaScript
  - Set reasonable LIMIT clauses for safety even on internal queries
  - Monitor query response sizes in production
- **Composite Indexes**: Always add indexes for:
  - Foreign key relationships
  - Columns used in `WHERE` clauses
  - Columns used in `ORDER BY` with `DISTINCT ON`
  - Multi-column patterns that match query patterns (e.g., `(agent_id, competition_id, timestamp DESC)`)
- **Query Optimization Principles**:
  - Prevent the linear query scaling problem: Instead of fetching a list then making separate queries per item, use joins or batch fetching
  - Use database cursors for large result sets
  - Leverage database window functions for ranking and analytics
  - Use `EXPLAIN ANALYZE` to verify query performance
  - Batch operations should process in chunks (e.g., 100 records at a time)

### Migration Best Practices
- **Migration Generation**: Always use `pnpm --filter api db:gen-migrations` after schema changes
- **Migration Naming**: Let Drizzle auto-generate names (e.g., `0041_mighty_wolverine.sql`)
- **Migration Review**: Always review generated SQL before committing
- **Backward Compatibility**: Migrations must be backward compatible for zero-downtime deployments
- **Data Migrations**: Use separate scripts in `apps/api/scripts/` for data migrations, never mix schema and data migrations
- **Rollback Strategy**: Document rollback SQL for destructive changes in PR description
- **Testing Migrations**: Test migrations on a copy of production data when possible
- **Index Creation**: Create indexes `CONCURRENTLY` in production to avoid table locks

### Database Type Safety
- **Function Return Types**: All functions must have explicit return types
  - Never rely on TypeScript inference for return types
  - Be especially explicit for async functions (e.g., `Promise<User[]>`)
- **Named Types**: Prefer named types/interfaces over inline definitions
  - Extract complex return types into named interfaces
  - Reuse types when the same shape appears multiple times
- **JSON Fields**: Never use `any` for jsonb columns
  - Create type guard functions for runtime validation
  - Use Zod schemas for complex JSON structures
  - Example: `isValidPerpsConfig(data): data is PerpsConfig`
- **Numeric Types**: Always parse database numerics to numbers
  - Use `Number()` or `parseFloat()` for numeric columns
  - Never pass numeric strings to frontend
  - Be explicit about precision requirements
- **Enum Safety**: Use TypeScript enums or literal unions for database enums
- **Null Handling**: Explicitly handle nullable columns with proper types
  - Use `| null` not `| undefined` for nullable database fields
  - Consider using Result types for operations that may fail

## Atomic Operations & Race Condition Prevention

### Service Layer Atomicity
- **TOCTOU Prevention**: Avoid Time-Of-Check-Time-Of-Use patterns where state can change between check and action
  - ❌ Bad: Check participant limit separately from adding agent (race condition):
    ```typescript
    // Multiple agents could pass this check simultaneously!
    const competition = await getCompetition(competitionId);
    if (competition.registeredParticipants < competition.maxParticipants) {
      await addAgentToCompetition(competitionId, agentId);  // ⚠️ Limit could be exceeded!
      await incrementParticipantCount(competitionId);
    }
    ```
  - ✅ Good: Use transactions with row locks (from `competition-repository.ts`):
    ```typescript
    await db.transaction(async (tx) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [recallnet/js-recall](https://github.com/recallnet/js-recall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
