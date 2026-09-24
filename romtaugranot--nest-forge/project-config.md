---
trigger: always_on
description: 1. **`index.ts` barrel files** — re-export from every module/folder so imports are clean (e.g. `import { Foo } from './foo'` not `import { Foo } from './foo/foo.service'`).
---

# Project Conventions

## Imports & Exports

1. **`index.ts` barrel files** — re-export from every module/folder so imports are clean (e.g. `import { Foo } from './foo'` not `import { Foo } from './foo/foo.service'`).
2. **No `import * as`** — always use named imports. If unavoidable, flag it to the user.
3. **Alias ambiguous import names** — alias to something descriptive, e.g. `import { resolve as resolvePath, dirname as getDirectoryName } from 'path'`.
4. **No default exports** — always use named exports. Default exports break barrel files and let import names drift from entity names.
5. **Domain module barrel files expose only the public API** — a domain module's top-level `index.ts` re-exports only what external consumers need: the module class, service(s), and shared types. Schemas, exceptions, refines, and data are implementation details and must not appear in the domain module barrel.
6. **NestJS `exports` array** — a provider (service) that will be injected by another module must appear in the module's `exports` array. Providers used only internally must not be exported.

## Folder Structure

Each entity type lives in its own dedicated folder:

| Entity | Folder | File suffix |
|---|---|---|
| Interfaces | `interfaces/` | `*.interface.ts` |
| Enums | `enums/` | `*.enum.ts` |
| Custom exceptions | `exceptions/` | `*.exception.ts` |
| Type aliases | `types/` | `*.type.ts` |
| Constants | `data/` | `*.data.ts` |
| Utilities | `utils/` | `*.util.ts` |
| Endpoint decorator bundles | `endpoints/` | `*.endpoint.ts` |
| Zod schemas | `schemas/` | `*.schema.ts` |
| Zod refine functions | `refines/` | `*.refine.ts` |
| Transform functions | `transforms/` | `*.transform.ts` |
| DTOs | `dtos/` | `*.dto.ts` |

7. **`common` folder is app-agnostic** — only put code in `common/` that is a pure library with zero knowledge of this application's domain.

## TypeScript Strictness

8. **Always annotate return types** — every function, method, and getter must have an explicit return type annotation. Never rely on inference for return types.
9. **No `any`** — if truly unavoidable, flag it explicitly to the user.
10. **No `as` (type assertions)** — if truly unavoidable, flag it explicitly to the user.
11. **No non-null assertion (`!`)** — handle nullability explicitly. If truly unavoidable, flag it to the user.
12. **`readonly` on immutable properties** — mark interface/class properties that should never be reassigned as `readonly`.
13. **`private readonly` for all constructor injections** — every constructor-injected dependency must be declared `private readonly`.
14. **Prefer `interface` over `type` for object shapes** — use `type` only for unions, intersections, and aliases.

## Code Style

15. **One entity per file** — never define more than one class, interface, or enum in a single file.
16. **`SCREAMING_SNAKE_CASE` for enum keys** — all enum member names must use `SCREAMING_SNAKE_CASE` (e.g. `MY_VALUE`, not `myValue`, `MyValue`, or `my-value`).
17. **Custom exceptions only** — always define and throw domain-specific exception classes; never throw plain `Error` or generic built-ins.
18. **Constants in `data/` only — one concern per file** — never hardcode constant values inline in logic or service files. Each `*.data.ts` file must contain constants that belong to a single cohesive topic, and the file name must precisely reflect that topic (e.g. `app-config-path.data.ts` holds only the config path constant). Never group unrelated constants in one file because they happen to share a module or prefix.
19. **Use destructuring** — use object and array destructuring when appropriate instead of repeated property access.
20. **Prefer getters over methods for read access — place them last** — whenever a method only reads and returns a value (no side effects, no parameters), use a `get` accessor instead. All `get` accessors must be placed after all other methods, at the bottom of the class body.
21. **No `function` keyword** — always use arrow functions (`const foo = () => {}`). Never use `function` declarations or expressions.
22. **No nested ternaries** — a ternary inside a ternary must be an `if/else` block. Keep ternaries flat and simple.
23. **`const` over `let` by default** — only use `let` when reassignment is genuinely needed.
24. **No reserved words as identifiers** — never use a JavaScript/TypeScript reserved word (e.g. `type`, `class`, `default`, `delete`, `in`, `new`, `return`, `throw`) as a variable, constant, or property name. Use a descriptive, domain-specific alternative instead (e.g. `credentialType` instead of `type`).
25. **No abbreviations** — always spell out identifiers in full. Never abbreviate (e.g. `error` not `err`, `config` not `cfg`, `response` not `res`, `request` not `req`). Only use abbreviations that are universally established conventions (e.g. `id`, `url`, `ssh`).
56. **No one-line `if` statements** — always use a block body (`{ }`) for `if`, `else if`, and `else` branches, even when the body is a single statement. Never write `if (condition) doSomething();` on one line.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [romtaugranot/nest-forge](https://github.com/romtaugranot/nest-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
