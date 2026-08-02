---
trigger: always_on
description: - Run all tests: `deno task test`
---

# GoatDB Testing Infrastructure

## Quick Start

- Run all tests: `deno task test`
- Debug specific test:
  `deno task test --suite=DatabaseFeature --test="should initialize" --deno-inspect-brk`
- Node.js only: `deno task test --runtime=node`

## Architecture Overview

The testing system is a custom, lightweight framework designed for
cross-platform compatibility:

- **Test runner**: @tests/run.ts - Orchestrates cross-platform execution
- **Test framework**: @tests/mod.ts - Provides TEST() function and TestSuite
  class
- **Entry point**: @tests/tests-entry.ts - Imports all test files
- **Node support**: @tests/node-run.ts - Handles TypeScript compilation for
  Node.js

### Key Design Principles

1. **No external dependencies** - Custom framework avoids third-party test
   runners
2. **Cross-platform** - Same tests run in Deno, Node.js, and browsers
3. **Sequential execution** - Tests run one at a time for consistency
4. **Resource management** - Automatic cleanup of temporary files/directories
5. **Simple API** - Just TEST() function and basic assertions
6. **Fast execution** - All tests run in single process

## Command Reference

### Basic Commands

| Command                               | Description                       |
| ------------------------------------- | --------------------------------- |
| `deno task test`                      | Run all tests in all environments |
| `deno task test --runtime=deno`       | Run tests in Deno only            |
| `deno task test --runtime=node`       | Run tests in Node.js only         |
| `deno test -A tests/specific.test.ts` | Run a specific test file directly |

### Debugging Commands

| Command              | Description                                  |
| -------------------- | -------------------------------------------- |
| `--deno-inspect-brk` | Attach Deno debugger (waits for debugger)    |
| `--node-inspect-brk` | Attach Node.js debugger (waits for debugger) |
| `--suite=NAME`       | Run only tests in specified suite            |
| `--test=NAME`        | Run only tests matching name                 |

### Environment Variables

| Variable       | Description                  |
| -------------- | ---------------------------- |
| `GOATDB_SUITE` | Filter to run specific suite |
| `GOATDB_TEST`  | Filter to run specific test  |

## Writing Tests

### CRITICAL: Setup Function Pattern

**MANDATORY STRUCTURE - All test files must follow exactly:**

```typescript
export default function setupMyTests() {
  TEST('suite', 'test-name', async (ctx: TestSuite) => {
    // test code here
  });
}
```

**BROKEN PATTERN - Never do this:**

```typescript
export default function setupMyTests() {
  // Empty function breaks test registration
}

TEST('suite', 'test', () => {}); // Outside setup = broken
```

**Rule: ALL TEST() calls must be inside the setup function.**

### Basic Test Structure

Every test file must:

1. Import TEST function from @tests/mod.ts
2. Import assertions from @tests/asserts.ts
3. Export a default `setup()` function that registers tests

See @tests/db.test.ts for a complete example of test structure.

### Available Assertions

All assertion functions are exported from @tests/asserts.ts:

- Boolean assertions: `assertTrue`, `assertFalse`
- Equality assertions: `assertEquals`, `assertNotEquals`
- Existence assertions: `assertExists`, `assertNotExists`
- Numeric comparisons: `assertLessThan`, `assertGreaterThan`, etc.
- Exception testing: `assertThrows`, `assertDoesNotThrow`
- Collection assertions: `expectToContain`

### Test Context Utilities

Each test receives a TestSuite context - see @tests/mod.ts:19-30 for available
methods:

- `tempDir(subPath?)` - Creates a temporary directory that's automatically
  cleaned up

## AI Agent Guidelines

**Test File Rules:**

1. **Setup function MUST contain all TEST() calls** - empty setup functions
   break registration
2. **Export setup as default** - `export default function setupX() { ... }`
3. **Import in tests-entry.ts** - Add `setupX()` call to main()
4. **Follow existing patterns** - Check similar test files first

**Common Failures:**

- Empty setup functions with TEST() calls outside
- Missing default export
- Setup function not called in tests-entry.ts

### When Adding Tests

1. **File placement**: Create test files in `/tests` with `.test.ts` suffix
2. **Export setup**: Every test file must export a default `setup()` function
3. **Descriptive names**: Use clear suite and test names that explain what's
   being tested
4. **Resource cleanup**: Always clean up resources in finally blocks
5. **Path format**: Remember GoatDB paths follow `/type/repo/item` format

### Test Organization

- Group related tests in the same suite name
- One concept per test - don't test multiple things
- Use consistent naming patterns across test files

### Common Test Patterns

#### Database Test Pattern

See @tests/db-trusted.test.ts:14-24 for database initialization tests. Key
points:

- Create database with `tempDir()` for isolation
- Always await `db.readyPromise()` before operations
- Clean up with `db.flushAll()` in finally block

#### Item Test Pattern

See @tests/db.test.ts:112-130 for item creation and update patterns:

- Use correct path format: `/type/repo/item`
- Register schemas before use

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goatplatform/goatdb](https://github.com/goatplatform/goatdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
