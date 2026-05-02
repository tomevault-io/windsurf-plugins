---
trigger: always_on
description: Error handling patterns for IPC communication between Rust backend and React frontend
---


# Error Handling Guide

This document describes the error handling patterns used in the LTK Manager application for communication between the Rust backend and React frontend through Tauri's IPC layer.

## Overview

The application uses a **typed Result pattern** for IPC communication, providing:
- Type-safe error codes for pattern matching
- Rich error context for debugging
- Consistent error handling across the entire application

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Rust Backend                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │   AppError   │ -> │ AppErrorResp │ -> │  IpcResult<T>    │   │
│  │  (internal)  │    │  (boundary)  │    │  (serialized)    │   │
│  └──────────────┘    └──────────────┘    └──────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │ JSON
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      TypeScript Frontend                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│  │  Result<T>   │ -> │   isOk/Err   │ -> │  UI Handling     │   │
│  │  (received)  │    │  (guards)    │    │  (toast/state)   │   │
│  └──────────────┘    └──────────────┘    └──────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

## Error Types

### Error Codes (Shared)

Error codes are shared between Rust and TypeScript:

```typescript
// @/utils/errors.ts
type ErrorCode =
  | "IO"                 // File system errors
  | "SERIALIZATION"      // JSON parsing errors
  | "MODPKG"             // Mod package errors
  | "LEAGUE_NOT_FOUND"   // League installation not found
  | "INVALID_PATH"       // Invalid file/directory path
  | "MOD_NOT_FOUND"      // Requested mod doesn't exist
  | "VALIDATION_FAILED"  // Input validation errors
  | "INTERNAL_STATE"     // Internal app state errors
  | "UNKNOWN";           // Unclassified errors
```

### AppError Interface

```typescript
// @/utils/errors.ts
interface AppError {
  code: ErrorCode;      // Machine-readable code
  message: string;      // Human-readable message
  context?: unknown;    // Optional contextual data
}
```

### Result Type

```typescript
// @/utils/result.ts
type Result<T, E = AppError> =
  | { ok: true; value: T }
  | { ok: false; error: E };
```

## Frontend Error Handling Patterns

### Pattern 1: Type Guards (Recommended for simple cases)

Use `isOk` and `isErr` type guards for simple conditional handling:

```typescript
import { api, isOk, isErr } from "@/lib/tauri";

async function loadMods() {
  const result = await api.getInstalledMods();
  
  if (isOk(result)) {
    setMods(result.value);
  } else {
    console.error("Failed to load mods:", result.error.message);
    showErrorToast(result.error.message);
  }
}
```

### Pattern 2: Match Function (Recommended for exhaustive handling)

Use `match` for cleaner handling of both cases:

```typescript
import { api, match } from "@/lib/tauri";

async function loadSettings() {
  const result = await api.getSettings();
  
  match(result, {
    ok: (settings) => {
      setSettings(settings);
    },
    err: (error) => {
      // Handle specific error codes
      if (error.code === "LEAGUE_NOT_FOUND") {
        showSetupWizard();
      } else {
        showErrorToast(error.message);
      }
    },
  });
}
```

### Pattern 3: Error Code Pattern Matching

Handle errors differently based on error codes:

```typescript
import { api, isErr } from "@/lib/tauri";

async function installMod(filePath: string) {
  const result = await api.installMod(filePath);
  
  if (isErr(result)) {
    switch (result.error.code) {
      case "INVALID_PATH":
        showError("The selected file doesn't exist or is inaccessible.");
        break;
      case "MODPKG":
        showError("The file is not a valid mod package.");
        break;
      case "VALIDATION_FAILED":
        showError("The mod package failed validation checks.");
        break;
      default:
        showError(`Installation failed: ${result.error.message}`);
    }
    return;
  }
  
  // Success case
  addModToLibrary(result.value);
  showSuccess("Mod installed successfully!");
}
```

## TanStack Query Integration

TanStack Query expects promises to **reject** on error for its error state to work properly. Since our `IpcResult` always resolves successfully (with errors encoded in the payload), we need adapter functions.

### Unwrapping Results for Queries

Create a utility to throw errors for TanStack Query:

```typescript
// @/utils/query.ts
import type { Result } from "./result";
import { isErr } from "./result";
import type { AppError } from "./errors";

/**
 * Unwrap a Result for use with TanStack Query.
 * Throws the error if Result is Err, allowing Query to catch it.
 */
export function unwrapForQuery<T>(result: Result<T>): T {
  if (isErr(result)) {
    throw result.error;
  }
  return result.value;
}

/**
 * Wrap an API call for use with TanStack Query.
 * Returns a function that throws on error.
 */
export function queryFn<T>(
  fn: () => Promise<Result<T>>
): () => Promise<T> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeagueToolkit/ltk-manager](https://github.com/LeagueToolkit/ltk-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
