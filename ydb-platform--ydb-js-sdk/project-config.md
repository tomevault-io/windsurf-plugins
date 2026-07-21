---
trigger: always_on
description: **Purpose:** This file contains rules and conventions for AI agents, IDE assistants, and developers working with this codebase. It defines how code should be written, tested, and maintained.
---

# Agent Instructions for YDB JavaScript SDK

**Purpose:** This file contains rules and conventions for AI agents, IDE assistants, and developers working with this codebase. It defines how code should be written, tested, and maintained.

## Our Principles

**Maintainability first** - This code will be maintained by others after you. Write for the next person, not for yourself. LLMs and AI agents will work with this code - keep it simple and clear.

**Simple over clever** - Choose elegant, straightforward solutions. Avoid deep nesting and excessive abstractions. If a concept is hard to explain, the implementation is probably wrong.

**Readability over micro-optimizations** - Code that's easy to understand is easier to extend and debug. Prioritize clarity over marginal performance gains.

**Explicit over implicit** - Type annotations in public APIs, clear variable names, meaningful comments that explain WHY. No magic, no surprises.

**Consistency over convenience** - Strict style rules (tabs, `let`, no semicolons) make code predictable. Consistency reduces cognitive load.

## Core Values

- **Easy to maintain:** Flat structure, clear naming, minimal abstractions. Future developers should grasp concepts quickly.
- **Easy to extend:** Adding features shouldn't require rewriting existing code. Modular design with explicit boundaries.
- **Reliable:** Every public API has tests. Type safety is non-negotiable. Errors are caught early, not at runtime.
- **Performant:** This is a database driver - efficiency matters. But not at the cost of code clarity.
- **Secure:** Never log credentials or tokens. Validate external inputs. Handle sensitive data carefully.

---

## Repository Structure

**Type:** npm workspaces monorepo
**Package Manager:** npm >= 10
**Runtime:** Node >= 20.19
**Build Tool:** Turbo
**Test Framework:** Vitest

### Package Architecture

```
@ydbjs/core
  └── depends on: api, auth, error, retry
      Core connection and driver functionality

@ydbjs/query
  └── depends on: core, value, error
      YQL query execution and transactions

@ydbjs/topic
  └── depends on: core, value, error
      Topic API (producers, consumers)

@ydbjs/value
  └── depends on: api
      YDB type system and values

@ydbjs/auth
  └── depends on: (minimal)
      Authentication (tokens, anonymous, metadata)

@ydbjs/retry
  └── depends on: error
      Retry policies and backoff strategies

@ydbjs/error
  └── depends on: api
      YDB error types and handling

@ydbjs/debug
  └── standalone (no internal deps)
      Centralized logging for all packages

@ydbjs/api
  └── standalone
      gRPC/Protobuf service definitions
```

**Important:** Breaking changes require coordination across dependent packages. Examples in `examples/` consume published packages, not local workspace versions

---

## Code Style Rules

### Formatting

- **Indentation:** Tabs (width: 4), always
- **Quotes:** Single quotes for strings
- **Semicolons:** None (semi: false)
- **Line length:** 120 characters max

**Exception:** Markdown and YAML files use 2 spaces

### Variable Declarations

```typescript
// DO: Always use let, even if never reassigned
let count = 0
let config = { timeout: 1000 }

// DON'T: Never use const
const value = 42 // ❌
```

### TypeScript

- Use `#` prefix for private fields
- Prefer explicit types in public APIs
- Enable strict mode

### Resource Cleanup

- Prefer `using`/`await using` over manual `afterEach`/shared-variable teardown
- Check for `[Symbol.dispose]`/`[Symbol.asyncDispose]` on the type before writing manual cleanup

```typescript
// DO
test('acquires and releases', async (tc) => {
  await using pool = new SessionPool(driver, { maxSize: 2 })
  let lease = await pool.acquire(tc.signal)
})

// DON'T
let pool: SessionPool | undefined
afterEach(async () => {
  await pool?.close()
  pool = undefined
})
```

### Comments

- Write in English, always
- Explain WHY, not WHAT
- Avoid obvious comments

```typescript
// DON'T: Obvious
let retries = 3 // Set retries to 3

// DO: Explains decision
let retries = 3 // AWS Lambda timeout is 3s, match retry window
```

---

## Testing Rules

### Structure

**CRITICAL:** Tests must be flat, never nested.

```typescript
// DO: Flat structure with test()
import { test, expect } from 'vitest'

test('processes string template', () => {
  let result = process('hello')
  expect(result).toBe('HELLO')
})

test('handles empty input', () => {
  let result = process('')
  expect(result).toBe('')
})

// DON'T: Nested describe/it blocks
describe('MyClass', () => {
  // ❌
  describe('method', () => {
    // ❌
    it('should work', () => {
      // ❌
      // ...
    })
  })
})
```

### Rules

- Use `test()` function only, never `it()`
- One test per `test()` call
- Import: `import { test, expect } from 'vitest'`

### Naming

- Be specific about scenario
- Use action verbs when helpful: `processes`, `handles`, `expects`, `creates`, `accepts`
- Never use "should" in test names
- No redundant context (don't repeat function name)

```typescript
// DO
test('processes string template')
test('handles single issue')
test('creates commit error')
test('stops when limit exceeded')


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ydb-platform/ydb-js-sdk](https://github.com/ydb-platform/ydb-js-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
