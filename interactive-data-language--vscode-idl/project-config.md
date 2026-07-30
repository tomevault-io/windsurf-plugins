---
trigger: always_on
description: TypeScript coding conventions for the vscode-idl codebase — naming, imports, types, null handling, functions, classes, error handling, JSDoc, formatting, and testing patterns
---


# TypeScript Coding Conventions

## Naming Conventions

**Functions (exported and internal file-private helpers):** PascalCase
```ts
export function CleanPath(uri: string): string { ... }
function BuildIdMap(nodes: ENVIModelerNode[]): Map<string, string> { ... }
export async function GetAutoComplete(...): Promise<GetAutoCompleteResponse> { ... }
```

**Classes:** PascalCase
```ts
export class GlobalIndex { ... }
export class CancellationToken { ... }
```

**Interfaces:** PascalCase with `I` prefix
```ts
export interface IFindTokensOptions { ... }
export interface IParserOptions { ... }
export interface IMessageToWorker<_Message extends string> { ... }
```

**Type aliases:** PascalCase, no `I` prefix
```ts
export type ParsedType = 'def' | 'notebook' | 'pro';
export type LogInterceptor = (options: ILogOptions) => void;
export type NameCounters = Record<string, number>;
```

**Module-level constants:** SCREAMING_SNAKE_CASE
```ts
export const LAYOUT_BASE_X = 1200;
export const TOKEN_TYPES: ITokenTypes = { BASIC: 0, START: 1, END: 2 };
```

**Exported mutable module-level state:** SCREAMING_SNAKE_CASE with `export let`
```ts
export let EXTENSION_FOLDER = '';
export let LANGUAGE_SERVER_CLIENT: LanguageClient;
```

**Local variables:** camelCase
```ts
const foundDefs: ITokenDef<TokenName>[] = [];
let hasLineContinuation = false;
```

**Generic type parameters:** PascalCase with
```ts
export class WorkerIO<Message extends string> { ... }
export interface IMessageToWorker<Message extends string> { ... }
```

**File naming:** kebab-case with dot-separated category suffix
- `name.class.ts` — class definitions
- `name.interface.ts` — interfaces, types, and constants
- `name.class.interface.ts` — interface for a specific class
- `name.spec.ts` — test files
- `verb-noun.ts` — standalone utility functions (e.g., `clean-path.ts`, `get-sort-idx.ts`)
- `index.ts` — barrel re-exports only

---

## Imports

**Always use named imports.** Default imports are not used.
```ts
import { CancellationToken } from '@idl/cancellation-tokens';
import { existsSync, readFileSync } from 'fs';
import { basename, dirname, join } from 'path';
```

**Star (`* as`) imports only** for modules without good typings or where the whole namespace is needed:
```ts
import * as vscode from 'vscode';
```

**CJS interop** uses `= require()` syntax:
```ts
import GlobToRegExp = require('glob-to-regexp');
```

**Use `@idl/...` path aliases** for all cross-library imports — never use deep relative paths across library boundaries:
```ts
import { IDL_TRANSLATION } from '@idl/translation';
import { CleanPath } from '@idl/shared/extension';
```

**Barrel `index.ts` files** contain only `export * from '...'` lines — no logic, no default exports, no selective re-exports:
```ts
export * from './lib/clean-path';
export * from './lib/simple-promise-queue.class';
```

---

## Type Annotations

**Use `undefined`, not `null`.** Never use `null` to represent the absence of a value — use `undefined` or optional (`?`) members instead. This keeps absence checks consistent and avoids the two-value problem (`null` vs `undefined`) throughout the codebase.

```ts
// DO — optional parameter (implicitly `string | undefined`)
export function MyFunc(optParam?: string): void { ... }

// DO — optional property in an interface
export interface IMyOptions {
  folder?: string;       // string | undefined, not string | null
  closer?: ITokenDef;
}

// DO — explicit undefined union when a variable may not be set yet
let current: string | undefined;

// DO — check with !== undefined (not truthiness) when falsy values are valid
if (current !== undefined) { ... }

// DON'T — never use null
let current: string | null = null;     // avoid
export function MyFunc(optParam: string | null): void { ... } // avoid
```

**Explicit parameter and return types on all exported functions:**
```ts
export function NextName(counters: NameCounters, prefix: string): string { ... }
export function ComputeLayout(nodes: ENVIModelerNode[]): Map<string, [number, number]> { ... }
```

Return type may be omitted when trivially inferred from a simple expression.

**`any` only when truly necessary** — e.g., cross-boundary serialization or genuinely unknown shapes. Do not use it to avoid typing work.

**Generics** are used extensively for typed message passing, event systems, and indexed lookups:
```ts
export class WorkerIO<Message extends string> { ... }
export type ExportedGlobalTokensByType = { [key in GlobalTokenType]: IGlobalIndexedToken<key>[] };
```

**Prefer `{ [key: string]: T }` index types over `Map`** for plain data objects that are serialized or iterated by key:
```ts
changedFiles: { [key: string]: boolean } = {};
globalTokensByFile: { [key: string]: GlobalTokens } = {};
```

**`Partial<Record<K, V>>`** and mapped types for optional lookup tables:
```ts
export const FIXED_DISPLAY_NAMES: Partial<Record<ENVIModelerNode['type'], string>> = { ... };
```

**`interface` vs `type`:**
- `interface` with `I` prefix → object shapes that describe a data structure or contract
- `type` → unions, mapped types, function signatures, string literal sets, and type composition

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [interactive-data-language/vscode-idl](https://github.com/interactive-data-language/vscode-idl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
