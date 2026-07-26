---
trigger: always_on
description: This file provides comprehensive guidance for AI coding assistants working with the team-labeler-action codebase.
---

# AGENT.md

This file provides comprehensive guidance for AI coding assistants working with the team-labeler-action codebase.

## Project Context

This is a GitHub Action that automatically labels pull requests and issues based on team membership. It's distributed as a compiled JavaScript action (Node 20) and must maintain backward compatibility with existing user configurations.

## Development Workflow

### Standard Development Cycle
```bash
# After making changes to src/
pnpm build        # Compile TS → JS (lib/)
pnpm test         # Verify tests pass
pnpm package      # Bundle with rollup (dist/)
```

### Quick Development
```bash
pnpm all          # Build + format + lint + package + test
```

### Testing Specific Scenarios
```bash
pnpm test -- __tests__/github.test.ts               # Single file
pnpm test -- -t "should return only the teams"       # Specific test
pnpm test:e2e                                         # End-to-end tests
```

## Code Patterns and Conventions

### Error Handling Pattern
```typescript
// src/main.ts pattern - always use core.error and core.setFailed
try {
  // action logic
} catch (error) {
  if (error instanceof Error) {
    core.error(error)
    core.setFailed(error.message)
  }
}
```

### GitHub API Client Pattern
```typescript
// Always create clients this way
const client = createClient(token)
const orgClient = orgToken ? createClient(orgToken) : null

// Check for null before using orgClient
if (!orgClient) {
  return []
}
```

### Pagination Pattern
```typescript
// CRITICAL: Always use paginate for lists that can exceed 100 items
const teams = await client.paginate(client.rest.teams.list, {
  org: orgName,
  per_page: 100
})

// For nested pagination (teams → members)
for (const team of teams) {
  const members = await client.paginate(
    client.rest.teams.listMembersInOrg,
    {
      org: orgName,
      team_slug: team.slug,
      per_page: 100
    }
  )
}
```

### Case-Insensitive Matching Pattern
```typescript
// teams.ts uses toLowerCase() for username/team comparisons
authors.some(a => a.toLowerCase() === author.toLowerCase())
```

**Note**: `src/github.ts` currently uses case-sensitive matching (`===`) when checking org team membership, while `src/teams.ts` correctly uses case-insensitive matching. This inconsistency should be kept in mind when working on team matching logic.

## File Structure and Responsibilities

### src/main.ts
- **Purpose**: Entry point, orchestrates the entire workflow
- **Key Responsibilities**: Input validation, client creation, calling helper functions, output setting
- **Pattern**: Uses early returns for missing context (no PR number, no author)
- **Important**: Must always call `core.setOutput('team_labels', ...)` for GitHub Actions output

### src/github.ts
- **Purpose**: All GitHub API interactions and context extraction
- **Key Functions**:
  - `getPrNumber()` / `getPrAuthor()`: Extract from `github.context.payload` (supports both PRs and issues)
  - `getLabelsConfiguration()`: Fetches YAML, supports external repos
  - `getUserTeamsWithDeps()`: Main team fetching logic with pagination (exported for testing)
  - `getUserTeams()`: Wrapper that injects `github.context.repo.owner` and `core` logger
  - `addLabels()`: Applies labels via `issues.addLabels` API
- **Testing Note**: `getUserTeamsWithDeps` accepts explicit dependencies for testability

### src/teams.ts
- **Purpose**: Pure team matching logic (no side effects)
- **Key Function**: `getTeamLabel()` - Maps author to team labels
- **Pattern**: Returns array (user can be in multiple teams)
- **Important**: Case-insensitive matching is required

### src/types.ts
- **Purpose**: Shared TypeScript types
- **Current Types**: `ExternalRepo` for cross-repo configuration

### Configuration Files
- `rollup.config.ts`: Bundler config (TypeScript → CommonJS bundle)
- `vitest.config.ts`: Unit test configuration
- `vitest.config.e2e.ts`: E2E test configuration
- `eslint.config.mjs`: ESLint flat config with TypeScript rules
- `.prettierrc.json`: Prettier formatting (no semi, single quotes, no trailing commas)

## Testing Patterns

### Test Framework
Tests use **vitest** (not jest). The API is largely compatible but uses `vi` instead of `jest` for mocking.

### Mock Structure
```typescript
// Standard mock pattern for GitHub client
const createMockClient = () => ({
  paginate: vi.fn(),
  rest: {
    teams: {
      list: vi.fn(),
      listMembersInOrg: vi.fn()
    }
  }
})

// Setup pagination mock
const setupPaginateMock = (mockClient, teamsResponse, membershipMap) => {
  mockClient.paginate.mockImplementation((method, params) => {
    if (method === mockClient.rest.teams.list) {
      return Promise.resolve(teamsResponse)
    }
    if (method === mockClient.rest.teams.listMembersInOrg) {
      const teamSlug = params.team_slug
      return Promise.resolve(membershipMap[teamSlug] || [])
    }
    return Promise.resolve([])
  })
}
```

### Test Organization
- Use **Given/When/Then** comments for clarity
- Group related tests in `describe` blocks by scenario
- Test pagination scenarios (100+ items)
- Test error handling with and without logger
- Test edge cases: no teams, no membership, API failures

### E2E Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JulienKode/team-labeler-action](https://github.com/JulienKode/team-labeler-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
