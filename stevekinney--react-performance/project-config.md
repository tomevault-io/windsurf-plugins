---
trigger: always_on
description: 1. **Zero-Duplication Doctrine**: If a utility already exists—use or extend it. Re-creating it is technical vandalism.
---


# TypeScript Best Practices

### Super Important 🚨

1. **Zero-Duplication Doctrine**: If a utility already exists—use or extend it. Re-creating it is technical vandalism.
2. **Mandatory Repo-Crawl Before Typing**: Ripgrep the codebase first. Clone code → public PR shaming.
3. **One-Purpose Functions**: One job per function. "And also…" means split it or delete it.
4. **Atomic Commits or Bust**: Each commit is a surgical strike. Unrelated changes = auto-reject.
5. **DRY or Die Tryin'**: Two copies = warning. Three copies = felony. CI fails on detectable duplication.
6. **Expand, Don't Explode**: Add behavior by extending existing utils—never fork a "v2".
7. **Simplicity Tax**: If reviewers need >30 sec to grok your PR, refactor until they don't.
8. **Comment Quotas Are Real**: If code needs a paragraph to explain, the code is wrong. Refactor.
9. **Kill Dead Code on Sight**: Delete unused paths/flags/TODO fossils before they multiply.
10. **Performance Is a Feature**: New code must meet or beat existing util speed. Slower = reject.
11. **Linter = Law**: Zero ESLint/Prettier warnings. "Stylistic" excuses ≠ defense.
12. **Context > Cleverness**: Readable beats wizardry. Explain it to a sleepy intern in <60 seconds.
13. **Guardrails > Guidelines**: Enforce via CI/git hooks. A rule that can't break the build is just cosplay.
14. **Fail Fast, Loud, Early**: Assertions everywhere. Silent fails are sabotage.
15. **Docs or It Didn't Happen**: Public utilities need JSDoc/TSDoc. Private helpers get inline types.
16. **Scope-Laser Mode**: Edit only the files absolutely required for the ticket. Touching >2 unrelated modules? Stop, commit nothing, and ping the human.

## Type Safety & Configuration

- Enable `strict: true` in @tsconfig.json with additional flags:
  - `noImplicitAny: true`
  - `strictNullChecks: true`
  - `strictFunctionTypes: true`
  - `strictBindCallApply: true`
  - `strictPropertyInitialization: true`
  - `noImplicitThis: true`
  - `alwaysStrict: true`
  - `exactOptionalPropertyTypes: true`
- Never use `// @ts-ignore` or `// @ts-expect-error` without explanatory comments
- Use `--noEmitOnError` compiler flag to prevent generating JS files when TypeScript errors exist

## Type Definitions

- Do not ever use `any`. Ever. If you feel like you have to use `any`, use `unknown` instead.
- Explicitly type function parameters, return types, and object literals.
- Please don't ever use Enums. Use a union if you feel tempted to use an Enum.
- Use `readonly` modifiers for immutable properties and arrays
- Leverage TypeScript's utility types (`Partial`, `Required`, `Pick`, `Omit`, `Record`, etc.)
- Use discriminated unions with exhaustiveness checking for type narrowing

## Advanced Patterns

- Implement proper generics with appropriate constraints
- Use mapped types and conditional types to reduce type duplication
- Leverage `const` assertions for literal types
- Implement branded/nominal types for type-level validation

## Code Organization

- Organize types in dedicated files (types.ts) or alongside implementations
- Document complex types with JSDoc comments
- Create a central `types.ts` file or a `src/types` directory for shared types

## Lint & TypeCheck Error Resolution (Lessons Learned)

### Import Path Management

- Always use path aliases (`@/` prefix) for src/ imports for consistency
- After refactoring, systematically update all import paths using search/replace
- Group imports: types first, then components, then utilities
- Use consistent import patterns across the codebase

### ESLint Rule Compliance

#### Nullish Coalescing (`??` vs `||`)

```typescript
// ❌ Problematic - catches falsy values like 0, false, ""
const result = value || defaultValue;

// ✅ Correct - only null/undefined trigger default
const result = value ?? defaultValue;
```

#### Type Import Consistency

```typescript
// ✅ Use proper type imports
import type { SomeType } from './types';

// ❌ Avoid import() type annotations
properties: (obj as Record<string, import('./types').SomeType>) ?? {};

properties: (obj as Record<string, SomeType>) ?? {};
```

#### No Explicit Any

```typescript
// ❌ Loses type safety
static tools = new Map<string, Tool<any, any>>();

// ✅ Use specific constraints
static tools = new Map<string, Tool<SchemaShape | undefined, Promise<unknown>>>();
```

### Chrome Extension API Types

- Chrome API Promise return types may not match @types definitions
- Use ESLint disable comments sparingly for known async APIs:

```typescript
// eslint-disable-next-line @typescript-eslint/await-thenable
const result = await chrome.tabs.detectLanguage(tabId);
```

### Error Handling in Tests

```typescript
// ❌ Unused error variable
try {
  testCode();
} catch (_error) {
  // error not used
}

// ✅ Bare catch when error details not needed
try {
  testCode();
} catch {
  // clean error handling
}
```

### Zod Version Compatibility

- Standardize on Zod v3 imported as `zod`
- Use standard imports: `import { z } from 'zod'`
- Define schema shapes consistently: `type SchemaShape = Record<string, z.ZodTypeAny>`

### Mock Implementation Strategy

- When services are deleted, create minimal mock implementations
- Use proper typing for mocks to maintain type safety

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stevekinney/react-performance](https://github.com/stevekinney/react-performance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
