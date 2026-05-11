---
trigger: always_on
description: This document provides guidelines for AI assistants working on the **fspec codebase**. This is about DEVELOPING fspec itself, not using it.
---

# Agent Development Guidelines for fspec

This document provides guidelines for AI assistants working on the **fspec codebase**. This is about DEVELOPING fspec itself, not using it.

---

## Project Overview

**fspec** is a standardized CLI tool for AI agents to manage Gherkin-based feature specifications and project work units using Acceptance Criteria Driven Development (ACDD).

- **Repository**: https://github.com/sengac/fspec
- **License**: MIT

For complete project context:
- **Project foundation**: [spec/FOUNDATION.md](spec/FOUNDATION.md)
- **Workflow**: run `fspec bootstrap` for complete details

---

## MANDATORY CODING STANDARDS - ZERO TOLERANCE

**ALL CODE MUST PASS QUALITY CHECKS BEFORE COMMITTING**

### CRITICAL DO NOT VIOLATIONS - CODE WILL BE REJECTED

#### TypeScript Violations:

- ❌ **NEVER** use `any` type - use proper types always
- ❌ **NEVER** use `as unknown as` - use proper type guards or generics
- ❌ **NEVER** use `require()` - only ES6 `import`/`export`
- ❌ **NEVER** use CommonJS syntax (`module.exports`, `__dirname`, `__filename`)
- ❌ **NEVER** use file extensions in TypeScript imports (`import './file.ts'` or `import './file.js'` → `import './file'`)
- ❌ **NEVER** use `var` - only `const`/`let`
- ❌ **NEVER** use `==` or `!=` - only `===` and `!==`
- ❌ **NEVER** skip curly braces: `if (x) doSomething()` → `if (x) { doSomething() }`

#### Import Violations:

- ❌ **NEVER** use dynamic imports unless absolutely necessary (e.g., `await import('./module')`)
- ❌ **NEVER** write: `import { Type } from './types'` when only using as type
- ✅ **ALWAYS** use static imports: `import { something } from './module'`
- ✅ **ALWAYS** write: `import type { Type } from './types'` for type-only imports
- ✅ **ALWAYS** omit file extensions in TypeScript imports - Vite handles the build

#### Interface Violations:

- ❌ **NEVER** use `type` for object shapes
- ✅ **ALWAYS** use `interface` for object definitions

#### Promise Violations:

- ❌ **NEVER** have floating promises - all promises must be awaited or explicitly ignored with `void`
- ❌ **NEVER** await non-promises

#### Variable Violations:

- ❌ **NEVER** declare unused variables
- ❌ **NEVER** use `let` when value never changes - use `const`

#### Console Violations:

- ❌ **NEVER** use `console.log/error/warn` in source code (tests are OK)
- ✅ **ONLY** use chalk for colored CLI output in commands

---

## MANDATORY IMPLEMENTATION PATTERNS

### ES Modules (Required):

```typescript
// ✅ CORRECT
import { fileURLToPath } from 'url';
import { dirname } from 'path';
const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

// ❌ WRONG
const __dirname = require('path').dirname(__filename);
```

### Type Safety (Required):

```typescript
// ✅ CORRECT
interface FeatureFile {
  path: string;
  tags: string[];
}
const feature: FeatureFile = loadFeature();

// ❌ WRONG
const feature: any = loadFeature();
```

### Error Handling (Required):

```typescript
// ✅ CORRECT - All async operations must have error handling
try {
  const result = await operation();
  return result;
} catch (error: any) {
  console.error(chalk.red('Error:'), error.message);
  throw error;
}
```

### CLI Output (Required):

```typescript
// ✅ CORRECT - Use chalk for colored output
import chalk from 'chalk';
console.log(chalk.green('✓ Feature file is valid'));
console.error(chalk.red('✗ Validation failed'));

// ❌ WRONG - Plain console.log in commands
console.log('Feature file is valid');
```

---

## File Organization

- **Keep files under 300 lines** - refactor when approaching this limit
- When a file exceeds 300 lines, stop and refactor BEFORE continuing
- Ask for approval before major refactoring

---

## Testing Requirements

- **Use Vitest exclusively** - NEVER use Jest
- **Write ALL tests in TypeScript** - NEVER create standalone JavaScript test files
- **NEVER write external JavaScript files for testing** - All tests must be TypeScript files running through Vitest
- **NEVER create .mjs or .js test files** - Only .ts test files within the project structure
- **NEVER test module imports using Node.js directly** - Always test through Vitest
- Write meaningful tests that verify actual functionality
- No trivial tests like `expect(true).toBe(true)`
- **Test Coverage:** All new code must have corresponding unit tests
- **Mock Patterns:** Use Vitest mocks, avoid actual file system in unit tests
- **Type Safety:** No `any` types allowed in tests - use proper type assertions

### Test File Requirements:

- ❌ **NEVER** create `test.mjs`, `test.js`, or any external JavaScript test files
- ❌ **NEVER** run tests with `node test.js` or `node test.mjs`
- ✅ **ALWAYS** create `.test.ts` or `.spec.ts` files
- ✅ **ALWAYS** run tests through `npm test` using Vitest
- ✅ **ALWAYS** import and test TypeScript modules directly in TypeScript test files

### Test Naming Convention:

```typescript
// Test file: src/commands/__tests__/validate.test.ts

describe('Feature: Gherkin Syntax Validation', () => {
  describe('Scenario: Validate single feature file with valid syntax', () => {
    it('should exit with code 0 and display success message', async () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sengac/fspec](https://github.com/sengac/fspec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
