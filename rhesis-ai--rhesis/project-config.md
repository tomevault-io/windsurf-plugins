---
trigger: always_on
description: TypeScript and ESLint rules that MUST be followed when creating, modifying, or reviewing any file under apps/frontend/, including .ts, .tsx, .js, and .jsx files. Also apply when discussing frontend linting, type safety, or ESLint configuration.
---


# Frontend TypeScript Linting Rules

## No Explicit `any`

The codebase enforces `@typescript-eslint/no-explicit-any` as a warning. **Never use `any` in new code.** Use `unknown` and narrow, or use the correct library/domain type.

### 1. Metadata and Generic Objects - Use `Record<string, unknown>`

```typescript
// BAD
interface MyEntity {
  metadata?: Record<string, any>;
  attributes: Record<string, any>;
}

// GOOD
interface MyEntity {
  metadata?: Record<string, unknown>;
  attributes: Record<string, unknown>;
}
```

Special cases where narrower types are appropriate:

```typescript
// HTTP headers are always strings
request_headers?: Record<string, string>;

// OpenTelemetry attributes
attributes: Record<string, string | number | boolean>;

// Known key-value config
auth?: Record<string, string | boolean | number>;
```

### 2. Catch Blocks - Use `unknown` with Type Narrowing

```typescript
// BAD
try {
  await api.fetch();
} catch (error: any) {
  setError(error.message);
}

// GOOD
try {
  await api.fetch();
} catch (error: unknown) {
  const message = error instanceof Error ? error.message : 'Unknown error';
  setError(message);
}
```

For accessing non-standard properties like `.status` or `.response`:

```typescript
} catch (error: unknown) {
  const errObj = error as Error & { status?: number; response?: { data?: { detail?: string } } };
  if (errObj.status === 404) {
    // handle not found
  }
  const message = errObj instanceof Error ? errObj.message : String(error);
}
```

### 3. MUI DataGrid Callbacks - Use Library Types

```typescript
import type { GridRenderCellParams, GridRowParams, GridCellParams, GridRowModel, GridColDef } from '@mui/x-data-grid';
import type { SxProps, Theme } from '@mui/material';

// BAD
columns: any[];
rows: any[];
onRowClick?: (params: any) => void;
getRowId?: (row: any) => string;
sx?: any;

// GOOD
columns: GridColDef[];
rows: GridRowModel[];
onRowClick?: (params: GridRowParams) => void;
getRowId?: (row: GridRowModel) => string;
sx?: SxProps<Theme>;
```

### 4. Type Assertions - Avoid `as any`

```typescript
// BAD
const result = response as any;
(theme.palette as any)[color];

// GOOD - use intermediate unknown when needed
const result = response as unknown as MyResponseType;
(theme.palette as unknown as Record<string, Record<string, string>>)[color];
```

When accessing window globals:

```typescript
// BAD
(window as any).myGlobal = value;

// GOOD
(window as Window & { myGlobal?: string }).myGlobal = value;
```

### 5. Function Return Types - Use Typed Promises

```typescript
// BAD
async function fetchData(): Promise<any> { ... }

// GOOD
async function fetchData(): Promise<Record<string, unknown>> { ... }

// BETTER - define a response interface
interface FetchResponse {
  data: MyEntity[];
  total: number;
}
async function fetchData(): Promise<FetchResponse> { ... }
```

### 6. Recharts and Chart Formatters

```typescript
// BAD
tickFormatter?: (value: any) => string;
tooltipFormatter?: (value: any, name: any) => string;

// GOOD
tickFormatter?: (value: string | number) => string;
tooltipFormatter?: (value: string | number, name: string) => string;
```

### 7. Test Files - Disable Per-File

Using `any` in test files for mocks and partial objects is acceptable. Add a file-level disable:

```typescript
/* eslint-disable @typescript-eslint/no-explicit-any */

import { render } from '@testing-library/react';
// ... test code using any for mocks
```

## Handling `unknown` in JSX

When using `Record<string, unknown>` types, `unknown` values can leak into JSX children through `&&` short-circuit operators, causing `TS2769: Type 'unknown' is not assignable to type 'ReactNode'`.

### 1. Extract and Narrow Before JSX

```typescript
// BAD - unknown leaks into JSX via &&
{test.metadata?.sources && Array.isArray(test.metadata.sources) && (
  <Grid>{/* TS error: unknown is not ReactNode */}</Grid>
)}

// GOOD - extract and narrow before JSX
const sources: Array<Record<string, string>> = Array.isArray(test.metadata?.sources)
  ? test.metadata.sources
  : [];

// Then in JSX:
{sources.length > 0 && (
  <Grid>{/* works fine */}</Grid>
)}
```

### 2. Guard Against Empty Objects `{}`

API responses typed as `Record<string, unknown>` may return `{}` where you expect a string or array. Always guard:

```typescript
// BAD - created_at might be {} not string
<span>{new Date(item.created_at).toLocaleDateString()}</span>

// GOOD
{typeof item.created_at === 'string' && (
  <span>{new Date(item.created_at).toLocaleDateString()}</span>
)}
```

### 3. Explicitly Type Boolean Conditions

```typescript
// BAD - isMultiTurn could be unknown, leaks into JSX children
{test.metadata?.is_multi_turn && <MultiTurnView />}

// GOOD - explicitly typed boolean
const isMultiTurn: boolean = Boolean(test.metadata?.is_multi_turn);
{isMultiTurn && <MultiTurnView />}
```

## Non-Null Assertions

The codebase enforces `@typescript-eslint/no-non-null-assertion` as a warning. Do not use the `!` postfix operator.

```typescript
// BAD
const name = user!.name;
const provider = providers.find(p => p.id === id)!;


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhesis-ai/rhesis](https://github.com/rhesis-ai/rhesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
