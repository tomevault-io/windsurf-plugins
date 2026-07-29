---
trigger: always_on
description: This document provides a comprehensive guide for an AI agent to effectively write, run, and manage unit and integration tests for a NextJS/TypeScript SDK project using Bun's built-in test runner.
---

# AGENTS.md

This document provides a comprehensive guide for an AI agent to effectively write, run, and manage unit and integration tests for a NextJS/TypeScript SDK project using Bun's built-in test runner.

---

## 🚀 Getting Started

Bun comes with a built-in, Jest-compatible test runner that's incredibly fast. To start, you don't need to install any extra packages. The test runner is part of the `bun` CLI.

### File Discovery

Bun's test runner automatically finds and executes tests in files that match the following patterns anywhere in your project:

- `*.test.{js|jsx|ts|tsx}`
- `*_test.{js|jsx|ts|tsx}`
- `*.spec.{js|jsx|ts|tsx}`
- `*_spec.{js|jsx|ts|tsx}`

### Test file locations

This codebase colocates test files with the code files, so dont put tests in separate **test** directories.

### Running Tests

Execute tests using the `bun test` command.

```shell
# Run all tests in the project
bun test

# Run all tests within a specific directory (by path fragment)
bun test <directory_name>

# Run a specific test file (by path fragment)
bun test <filename_fragment>

# Run a specific test file by its exact path
bun test ./tests/specific-file.test.ts
```

---

## ✍️ Writing Tests

Bun's test runner is designed to be a drop-in replacement for Jest. It uses a familiar `describe`, `test`, and `expect` API.

### Basic Test Structure

Tests are defined with the `test()` function (or its alias `it()`) and grouped into suites with `describe()`. Assertions are made using `expect()`.

```typescript
// Import test utilities from bun:test
import { test, expect, describe } from "bun:test";

describe("SDK Math Utilities", () => {
  // A simple synchronous test
  test("should add two numbers correctly", () => {
    expect(2 + 2).toBe(4);
  });

  // An asynchronous test using async/await
  test("should resolve a promise", async () => {
    const result = await Promise.resolve("hello");
    expect(result).toEqual("hello");
  });
});
```

### Assertions with `expect`

Bun implements the full Jest `expect` API. Here are some common matchers:

- `.toBe(value)`: Strict equality (`===`).
- `.toEqual(value)`: Deep equality for objects and arrays.
- `.toThrow(error?)`: Checks if a function throws an error.
- `.toHaveBeenCalled()`: For checking if a mock function was called.
- `.toHaveBeenCalledWith(...args)`: Checks arguments passed to a mock.
- `.toMatchSnapshot()`: Performs snapshot testing.
- `.toMatchInlineSnapshot()`: Performs inline snapshot testing.

You can also verify that a certain number of assertions were called, which is useful in asynchronous code.

```typescript
test("should run a specific number of assertions", () => {
  expect.hasAssertions(); // Ensures at least one assertion is called
  expect.assertions(2); // Ensures exactly two assertions are called

  expect(1).toBe(1);
  expect(true).not.toBe(false);
});
```

### Parametrized Tests with `.each`

Run the same test logic with different data using `test.each` or `describe.each`. This is ideal for data-driven testing.

```typescript
const additionCases = [
  [1, 2, 3],
  [0, 0, 0],
  [-5, 5, 0],
];

test.each(additionCases)("add(%i, %i) should equal %i", (a, b, expected) => {
  expect(a + b).toBe(expected);
});
```

---

## 🧪 Test Database & Mock Data

For integration tests that involve database queries, the project provides helper functions to ensure a consistent and isolated testing environment.

### Test Database Setup

Use the `setupTestDb` function from `src/__testing__/helpers/db.ts` to create a fresh, in-memory SQLite database for your test suites. This function handles schema migrations automatically.

**Pattern:** Instantiate the database at the beginning of your `describe` block.

```typescript
import { describe, test, expect } from "bun:test";
import { setupTestDb } from "@/src/__testing__/helpers/db";

describe("Database-Related Feature", () => {
  const db = setupTestDb();
  // ... tests that use db instance
});
```

### Mock Data Generation

Use the mock data generators from `src/__testing__/helpers/mock-data.ts` (e.g., `generateMockUsers`, `generateMockPullRequests`) to populate your test database. These functions use `@faker-js/faker` to produce realistic data and allow you to override any fields for specific test cases.

### Complete Test Example

Follow this pattern for writing tests that interact with the database. This ensures that tests are self-contained, repeatable, and easy to understand.

```typescript
import { describe, test, expect } from "bun:test";
import { setupTestDb } from "@/src/__testing__/helpers/db";
import {
  generateMockUsers,
  generateMockPullRequests,
} from "@/src/__testing__/helpers/mock-data";
import * as schema from "@/lib/data/schema";
import { getRepositoryContributors } from "./queries"; // The function being tested

describe("Repository Queries", () => {
  const db = setupTestDb();

  test("should return unique contributors for a given repository", async () => {
    // 1. Arrange: Insert mock data
    const users = generateMockUsers([
      { username: "user-a" },
      { username: "user-b" },
    ]);
    await db.insert(schema.users).values(users);


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elizaOS/elizaos.github.io](https://github.com/elizaOS/elizaos.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
