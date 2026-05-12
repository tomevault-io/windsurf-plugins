---
trigger: always_on
description: Apply when writing TypeScript code. Strict types, discriminated unions, async patterns, and runtime safety.
---


# Sub-Skill: TypeScript Best Practices
<!-- target: ~2500 tokens (real tiktoken count) | 17 rules with severity classification -->

**Purpose:** Prevents the TypeScript-specific mistakes LLMs make repeatedly — weak types, unsafe assertions, and patterns that compile but fail at runtime.

## Rule classification

- **MUST** — load-bearing. Violating leaks runtime errors past the type checker. Never break.
- **SHOULD** — default behavior. Deviation needs a documented reason in the code or PR.
- **AVOID** — usually wrong; documented exception inline where needed.

**Where these rules don't strictly apply:** test fixtures, generated types (e.g. from GraphQL codegen, OpenAPI generators, Prisma), declaration files (`*.d.ts`) for untyped third-party libraries, and migration scripts may legitimately differ. The rules below apply to **production application code**.

---

## Type Safety

1. **MUST: Never use `any`. Use `unknown` and narrow it.** `any` disables the type checker entirely. `unknown` forces you to prove the type before use. *Exception: third-party libraries without types and explicit dynamic-data boundaries (e.g. JSON parse at the API edge), with a comment explaining why.*
   ```ts
   // Wrong
   function parse(data: any) { return data.name; }

   // Correct
   function parse(data: unknown): string {
     if (typeof data === 'object' && data !== null && 'name' in data) {
       return String((data as { name: unknown }).name);
     }
     throw new Error('Invalid data shape');
   }
   ```

2. **AVOID: `Object` or `{}` as a type.** Both accept nearly everything. Use `Record<string, unknown>` for arbitrary objects or define an explicit interface.
   ```ts
   // Wrong
   function merge(a: {}, b: Object): {} { ... }

   // Correct
   function merge<T extends Record<string, unknown>>(a: T, b: Partial<T>): T { ... }
   ```

3. **SHOULD: Use `as` only when you know more than the compiler — and document why.** Prefer type guards or `satisfies` instead.
   ```ts
   // Wrong — silences the error, hides the bug
   const user = response.data as User;

   // Correct — validate first
   function isUser(v: unknown): v is User {
     return typeof v === 'object' && v !== null && 'id' in v && 'email' in v;
   }
   const user = isUser(response.data) ? response.data : null;
   ```

4. **SHOULD: Mark immutable data `readonly`.** Prevents accidental mutation and communicates intent.
   ```ts
   // Avoid
   function process(ids: string[]) { ids.push('extra'); }

   // Prefer
   function process(ids: readonly string[]) { /* ids.push() is a compile error */ }
   ```

5. **MUST: Enable `strictNullChecks` and handle every `T | undefined`.** Optional chaining `?.` returns `undefined` — always handle that branch.
   ```ts
   // Wrong
   const name = user?.profile.name.toUpperCase(); // crashes if name is undefined

   // Correct
   const name = user?.profile.name?.toUpperCase() ?? 'Anonymous';
   ```

6. **SHOULD: Use branded types for IDs.** Prevents passing a `UserId` where an `OrderId` is expected — both are `string` at runtime.
   ```ts
   type UserId = string & { readonly _brand: 'UserId' };
   type OrderId = string & { readonly _brand: 'OrderId' };

   function createUserId(raw: string): UserId { return raw as UserId; }

   function getUser(id: UserId): User { ... }
   // getUser(orderId) → compile error
   ```

---

## Patterns

7. **SHOULD: Use discriminated unions for state, not optional fields.** Optional fields force you to reason about all combinations. A discriminated union makes illegal states unrepresentable.
   ```ts
   // Avoid — 8 possible combinations, most invalid
   type Request = { loading?: boolean; data?: User; error?: Error };

   // Prefer — exactly 3 valid states
   type Request =
     | { status: 'idle' }
     | { status: 'loading' }
     | { status: 'success'; data: User }
     | { status: 'error'; error: Error };
   ```

8. **SHOULD: Use `satisfies` to validate shape without widening the type.** `as const` preserves literals; `satisfies` validates against an interface without losing them.
   ```ts
   const config = {
     host: 'localhost',
     port: 5432,
   } satisfies DatabaseConfig;
   // config.port is still typed as 5432, not number
   ```

9. **SHOULD: Use `const` objects instead of `enum`.** Enums emit runtime code, have surprising reverse-mapping behavior, and are not idiomatic TypeScript.
   ```ts
   // Avoid
   enum Direction { Up, Down, Left, Right }

   // Prefer
   const Direction = { Up: 'Up', Down: 'Down', Left: 'Left', Right: 'Right' } as const;
   type Direction = typeof Direction[keyof typeof Direction];
   ```

10. **AVOID: Barrel `index.ts` re-exports in large modules.** They cause circular dependency chains that are hard to debug. Export directly from source files or use explicit named re-exports only.
    ```ts
    // Wrong — index.ts re-exports everything, A imports B through index, B imports A through index
    export * from './userService';
    export * from './orderService';

    // Correct — import directly
    import { getUser } from './services/userService';
    ```

---

## Error Handling


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sordi-ai/skill-everything](https://github.com/sordi-ai/skill-everything) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
