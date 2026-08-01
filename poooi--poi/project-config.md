---
trigger: always_on
description: This document provides instructions and context for AI agents working on the poi repository.
---

# AI Agent Instructions for poi Repository

This document provides instructions and context for AI agents working on the poi repository.

## Repository Overview

poi is an Electron-based game assistant for Kantai Collection (KanColle). It uses:

- **React** for UI components
- **Redux** with **@reduxjs/toolkit** for state management
- **TypeScript** for type safety
- **Jest** for testing
- **ESLint** with Prettier for code formatting
- **npm** as the package manager

## Project Structure

### Key Directories

- `/views/redux/` - Redux store, reducers, and actions
  - `/views/redux/info/` - Info reducers for game state (ships, fleets, equips, etc.)
  - `/views/redux/actions.ts` - Action creators for API responses
- `/views/components/` - React components
- `/views/utils/` - Utility functions
- `/lib/` - Core library code (Electron main process)

### Test Location

Tests are located in `__tests__` directories adjacent to the code being tested:

- `/views/redux/info/__tests__/` - Tests for info reducers
- Test files use `.spec.ts` or `.spec.es` extension

## TypeScript Migration Guidelines

### File Extensions

The codebase uses multiple file extensions:

- `.ts` - TypeScript files (preferred for new code)
- `.tsx` - TypeScript React components
- `.es` - ES6 JavaScript files (legacy, being migrated)
- `.js` - JavaScript files

### Migrating `.es` files to TypeScript

1. Rename the file from `.es` to `.ts`
2. Add type annotations for:
   - Function parameters and return types
   - State interfaces
   - Action interfaces
3. Export type definitions for use by other modules
4. Update imports in dependent files if needed

### Example State Interface Pattern

```typescript
export interface Ship {
  api_id: number
  api_ship_id?: number
  api_nowhp?: number
  api_maxhp?: number
  // ... other properties
}

export interface ShipsState {
  [key: string]: Ship
}
```

### Example Reducer Pattern

```typescript
export function reducer(
  state: ShipsState = {},
  { type, body, postBody }: Action,
  store?: Store,
): ShipsState {
  switch (type) {
    case '@@Response/kcsapi/api_port/port':
      // handle action
      return newState
    default:
      return state
  }
}
```

## API Action Creators

### Location

API action creators are defined in `/views/redux/actions.ts`.

### kcsapi Package

The `kcsapi` package provides TypeScript types for the game API:

- Request types: `API*Request` (e.g., `APIGetMemberDeckRequest`)
- Response types: `API*Response` (e.g., `APIGetMemberDeckResponse`)

### Pattern for Creating Action Creators

```typescript
import { createAction } from '@reduxjs/toolkit'
import { APIExampleRequest, APIExampleResponse } from 'kcsapi'

interface GameResponsePayload<Body, PostBody> {
  method: string
  path: string
  body: Body
  postBody: PostBody
  time: number
}

export const createAPIExampleResponseAction = createAction<
  GameResponsePayload<APIExampleResponse, APIExampleRequest>
>('@@Response/kcsapi/api_path/endpoint')
```

### Payload Shape Notes

- Some endpoints return arrays even if `kcsapi` exports an item type (e.g. `api_get_member/ndock` is `APIGetMemberNdockResponse[]` in practice). Prefer matching the real response shape when typing `GameResponsePayload`.
- Avoid dangerous double assertions like `as unknown as T` in reducers/middlewares.
  - Prefer typing at the action creator boundary (`views/redux/actions.ts`) and carrying real types through.
  - If the real payload is known to be partial/variant, introduce a small `*Compat` type (e.g. `Partial<APIShip> & { api_id: number }`) and use it consistently.
  - If you must assert, do it once at the boundary and keep internal logic strongly typed.

### Finding Available Types

To see available types from kcsapi:

```bash
cat node_modules/kcsapi/index.ts
```

### Field Name Reference

- For game API field naming and rough payload shape reference, `ElectronicObserver/Other/Information/apilist.txt` is often useful (may be outdated; treat as a hint, not a source of truth).
- URL: `https://raw.githubusercontent.com/andanteyk/ElectronicObserver/develop/ElectronicObserver/Other/Information/apilist.txt`

### Custom Types for Missing APIs

Some API endpoints are not typed in kcsapi. Define custom types with a `FIXME` comment:

```typescript
// FIXME: Not in kcsapi package - @@Response/kcsapi/api_req_hensei/preset_order_change
export interface APIReqHenseiPresetOrderChangeRequest {
  api_verno: string
  api_preset_from: string
  api_preset_to: string
}

export interface APIReqHenseiPresetOrderChangeResponse {
  api_result: number
  api_result_msg: string
}
```

### Currently Missing from kcsapi

These API endpoints are used but not typed in the kcsapi package:

1. `@@Response/kcsapi/api_req_hensei/preset_order_change`
2. `@@Response/kcsapi/api_req_member/updatedeckname`
3. `@@Response/kcsapi/api_req_air_corps/change_name`
4. `@@Response/kcsapi/api_req_air_corps/change_deployment_base`

## Testing Guidelines

### Running Tests

```bash
# Run all tests
npm test

# Run specific tests
npm test -- --testPathPattern="info"

# Run with coverage
npm test -- --coverage
```

### Test File Pattern

```typescript
import { reducer, StateType } from '../reducer-file'

describe('reducer name', () => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poooi/poi](https://github.com/poooi/poi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
