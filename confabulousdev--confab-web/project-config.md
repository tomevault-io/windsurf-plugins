---
trigger: always_on
description: Backend API is documented in `backend/API.md`. **Keep this file up to date** when modifying API endpoints, request/response schemas, or authentication.
---

# Confab Development Notes

## API Documentation

Backend API is documented in `backend/API.md`. **Keep this file up to date** when modifying API endpoints, request/response schemas, or authentication.

## Documentation Maintenance

When changing code, update the corresponding package/module README. Key things to keep current: file lists, exported API descriptions, invariants, dependency lists, and extension checklists. If a change spans multiple packages, also check the index READMEs (`backend/internal/README.md`, `frontend/src/README.md`) and `CLAUDE.md`. Documentation that contradicts the code is worse than no documentation.

## Development Process

**Follow this workflow for all implementation tasks:**

### 1. Plan First

Before writing any code, create a clear plan:
- Use the TodoWrite tool to break down the task into concrete implementation steps
- Consider edge cases, error handling, and potential impacts on existing code
- For Linear tickets, update the issue with the plan before starting implementation

### 2. Avoid Code Duplication (DRY)

**Before implementing any logic, check if similar logic already exists elsewhere.**

#### Common Duplication Patterns to Avoid

1. **Query logic duplicated in SQL and Go** - If you have staleness/validity checks, they should exist in ONE place. Either SQL calls a shared function, or Go is the single source of truth.

2. **Utility functions copied between packages** - Functions like `extractAgentID`, `mergeChunks`, `parseChunkKey` should live in ONE shared location and be imported everywhere.

3. **Business logic in multiple code paths** - If both "on-demand" and "background worker" paths do the same thing, extract the shared logic to a common function.

#### Where Shared Code Lives

- **Chunk operations** (download, merge, parse keys): `internal/storage/chunks.go`
- **Analytics computation**: `internal/analytics/` package
- **Card staleness validation**: `internal/analytics/cards.go` (`IsValid`, `AllValid` methods)
- **Smart recap generation**: `internal/analytics/smart_recap_generator.go`

#### Before Writing New Code

1. Search for existing implementations: `grep -r "functionName" backend/`
2. Check if a similar pattern exists in related code paths
3. If you find duplication, refactor to a shared location FIRST
4. Add comments noting where the shared logic lives (e.g., "This mirrors AllValid() in cards.go")

### 3. Test Coverage

Every change should include appropriate tests. **Insufficient test coverage is not acceptable.**

#### What to Test

1. **Unit tests** for pure logic and helper functions:
   - Data transformation functions
   - Validation logic
   - Parsing/formatting utilities
   - Business rule calculations

2. **Integration tests** for database operations:
   - SQL queries (especially complex ones with JOINs, CTEs, aggregations)
   - CRUD operations and edge cases
   - Constraint violations and error handling
   - Use `testutil.SetupTestEnvironment(t)` for containerized Postgres/MinIO

3. **API tests** for HTTP handlers:
   - Success paths with valid input
   - Error responses for invalid input
   - Authentication/authorization checks
   - Edge cases (empty results, pagination bounds)

#### Test Coverage Checklist

Before presenting work, verify you have tests for:

- [ ] **Happy path**: Does the feature work correctly with valid input?
- [ ] **Edge cases**: Empty inputs, boundary values, nil/null handling
- [ ] **Error cases**: Invalid input, missing data, permission denied
- [ ] **SQL queries**: If you wrote SQL, test it with real data (integration test)
- [ ] **Configuration**: If you added config options, test parsing and validation

#### Test Patterns in This Codebase

```go
// Unit test (runs with -short)
func TestHelperFunction(t *testing.T) {
    // Test pure logic
}

// Integration test (requires Docker, skipped with -short)
func TestDatabaseOperation(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test")
    }
    env := testutil.SetupTestEnvironment(t)
    env.CleanDB(t)
    // Test with real database
}
```

#### When to Ask About Test Coverage

If implementing a feature without tests, pause and ask:
- "What test cases would give us confidence this works?"
- "Are there edge cases I should test?"
- "Should this have integration tests for the SQL queries?"

### 4. Self-Review Before Presenting

**Before presenting any result to the human, perform a thorough code review:**

1. **Re-read all modified files directly** - Use the Read tool to review each changed file. Do not rely solely on memory or tests passing. Actually read the code again with fresh eyes.

2. **Review critically, as if reviewing someone else's work:**
   - Check for bugs, edge cases, and error handling gaps
   - Look for logic errors and off-by-one mistakes
   - Verify interactions between modified components work correctly
   - Check that conditional logic handles all cases (especially error/null states)

3. **Verify code quality:**
   - Follows existing patterns and conventions in the codebase
   - No debug code, TODOs, or incomplete implementations remain
   - No security vulnerabilities introduced


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ConfabulousDev/confab-web](https://github.com/ConfabulousDev/confab-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
