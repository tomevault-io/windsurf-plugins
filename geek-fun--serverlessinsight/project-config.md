---
trigger: always_on
description: This document defines coding conventions that AI agents (and human developers) must follow when contributing to ServerlessInsight.
---

# AGENTS.md - Coding Conventions for AI Agents

This document defines coding conventions that AI agents (and human developers) must follow when contributing to ServerlessInsight.

## Build/Lint/Test Commands

```bash
# Build the project
npm run build

# Lint check (without fixing)
npm run lint:check

# Lint and auto-fix issues
npm run lint:fix

# Run all tests
npm test

# Run tests for a specific file pattern
npm test -- --testPathPatterns="fc3Resource" --passWithNoTests

# Run a single test file
npm test -- --testPathPatterns="path/to/test.test.ts" --passWithNoTests

# Run tests matching a test name pattern
npm test -- --testNamePattern="should create resource" --passWithNoTests

# CI test run (no debug output)
npm run test:ci
```

## Internationalization (i18n) - MANDATORY

**All user-facing messages MUST use the i18n system. Never hardcode English strings.**

### Pattern

1. Add the message key to `src/lang/en.ts`:

   ```typescript
   YOUR_MESSAGE_KEY: 'Your message with {{variable}} placeholder',
   ```

2. Add the Chinese translation to `src/lang/zh-CN.ts`:

   ```typescript
   YOUR_MESSAGE_KEY: '带有 {{variable}} 占位符的中文消息',
   ```

3. Use the message in code:

   ```typescript
   import { lang } from '../../lang';

   throw new Error(lang.__('YOUR_MESSAGE_KEY', { variable: value }));
   ```

### Examples

**WRONG - Hardcoded English string:**

```typescript
throw new Error(`RAM role "${roleName}" does not exist in the cloud provider.`);
```

**CORRECT - Using i18n:**

```typescript
import { lang } from '../../lang';

throw new Error(lang.__('RAM_ROLE_NOT_FOUND_IN_CLOUD', { roleName }));
```

### Where this applies

- Error messages
- Warning messages
- Info/log messages
- User prompts
- Any text that could be seen by an end user

### Where this does NOT apply

- Internal debug identifiers
- API request/response field names
- Code comments
- Variable/function names

## Functional TypeScript

### Function Style

- **Define functions as `const`**: Use `const xxx = (...) => ...` syntax
- **Prefer functional decomposition over OOP**: Avoid classes unless strictly necessary
- **Keep functions small and focused**: Single responsibility, easy to test

```typescript
// GOOD
const buildAssumeRolePolicy = (trustedServices: string[]): string =>
  JSON.stringify({
    Version: '1',
    Statement: [{ Action: 'sts:AssumeRole', Effect: 'Allow', Principal: { Service: trustedServices } }],
  });

// AVOID
class RolePolicyBuilder {
  build(trustedServices: string[]) { ... }
}
```

### Declarative Collection Handling

Replace `for`/`while` loops with `map`, `filter`, `find`, `some`, `every`, `reduce`, `flatMap`. Favor pipeline-style transformations.

```typescript
// GOOD
const roleArns = instances.filter((i) => i.type === 'ALIYUN_RAM_ROLE').map((i) => i.roleArn);

// AVOID
const roleArns: string[] = [];
for (const i of instances) {
  if (i.type === 'ALIYUN_RAM_ROLE') {
    roleArns.push(i.roleArn);
  }
}
```

### Immutability

- Avoid in-place mutation (`push`, `splice`, mutating objects/arrays)
- Return new arrays/objects
- Model changes as explicit state-transform functions

```typescript
// GOOD - return new object
const newState = { ...state, resources: { ...state.resources, [id]: resource } };

// AVOID - mutation
state.resources[id] = resource;
```

### Pure Functions

- Keep functions side-effect-free where possible
- Isolate effects (I/O, logging) at boundaries
- Keep core logic pure

## Module & Export Discipline

### Module Boundaries

- Each module exports **only** via its `index.ts`
- Avoid deep imports from outside the module

```typescript
// GOOD
import { createAliyunClient } from '../../common/aliyunClient';

// AVOID
import { createRamOperations } from '../../common/aliyunClient/ramOperations';
```

### Export Discipline

- Only export functions/types/constants that are used outside the module
- Keep implementation details private

### Provider-Agnostic Design

- Keep abstractions provider-agnostic
- Clean separation of concerns between providers

## Code Style

### TypeScript

- **Target**: ES2020, CommonJS modules
- **Strict mode**: Enabled
- **Use `type` over `interface`** for type definitions
- Use early returns to reduce nesting

### Imports

Group imports in this order, separated by blank lines:

```typescript
// 1. External packages
import RamClient from '@alicloud/ram20150501';
import * as ram from '@alicloud/ram20150501';
import fs from 'node:fs';

// 2. Internal modules (use relative paths with barrel exports)
import { createAliyunClient } from '../../common/aliyunClient';
import { Context, StateFile } from '../../types';

// 3. Types (if needed separately)
import type { RamRoleInfo } from './types';
```

### Naming Conventions

- **Files**: camelCase for modules (e.g., `fc3Resource.ts`, `ramOperations.ts`)
- **Functions**: camelCase, verb-first (e.g., `createResource`, `getRoleArnFromState`)
- **Types**: PascalCase (e.g., `FunctionDomain`, `StateFile`)
- **Constants**: SCREAMING_SNAKE_CASE (e.g., `RAM_ROLE_PROPAGATION_DELAY_MS`)
- **Private helpers**: camelCase at module level (e.g., `buildAssumeRolePolicy`, `delay`)

### Error Handling

- Always use i18n for error messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geek-fun/serverlessinsight](https://github.com/geek-fun/serverlessinsight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
