---
trigger: always_on
description: **AI Agent Documentation for sveltekit-superforms**
---

# AGENTS.md

**AI Agent Documentation for sveltekit-superforms**

This document provides essential information for AI agents (like GitHub Copilot, Claude, ChatGPT, etc.) working with the sveltekit-superforms codebase. Last updated: October 18, 2025.

---

## Project Overview

**sveltekit-superforms** is a comprehensive form management library for SvelteKit that provides:

- Server and client-side validation with 12+ validation libraries
- Progressive enhancement (works without JavaScript)
- Type-safe form handling with automatic TypeScript inference
- Advanced features: nested data, file uploads, multiple forms, SPA mode, snapshots, tainted detection

**Website:** https://superforms.rocks/  
**Repository:** https://github.com/ciscoheat/sveltekit-superforms  
**License:** MIT

---

## Architecture & Core Concepts

### 1. **Validation Adapters**

The library uses a **validation adapter pattern** to support multiple validation libraries. Each adapter lives in `src/lib/adapters/`.

**Supported Libraries:**

- Zod (v3 via `zod.ts`, v4 via `zod4.ts`)
- Valibot, Yup, Joi, ArkType, Effect, TypeBox, Superstruct, VineJS
- class-validator, schemasafe, simple-schema
- JSON Schema (direct)

**Adapter Interface** (`src/lib/adapters/adapters.ts`):

```typescript
interface ValidationAdapter<Out, In> {
	validate(data): Promise<ValidationResult<Out>>;
	jsonSchema?: JSONSchema;
	defaults?: Out;
	constraints?: InputConstraints<Out>;
}
```

**Key Files:**

- `src/lib/adapters/adapters.ts` - Core adapter types and factory
- `src/lib/adapters/typeSchema.ts` - Type-level adapter definitions
- `src/lib/adapters/zod4.ts` - Zod v4 adapter (has special JSON Schema overrides for date/bigint)

### 2. **Core API Surface**

**Server-side (`src/lib/superValidate.ts`):**

```typescript
superValidate(data?, adapter, options?) -> SuperValidated<T>
```

- Validates data against schema
- Returns form object with `{ id, valid, posted, errors, data, constraints, message }`
- Can accept: RequestEvent, Request, FormData, URLSearchParams, URL, Partial<T>, null, undefined

**Client-side (`src/lib/client/superForm.ts`):**

```typescript
superForm(data: SuperValidated<T>, options?: FormOptions<T>) -> SuperForm<T>
```

- Creates reactive form stores
- Returns: `{ form, errors, constraints, message, tainted, delayed, enhance, ... }`
- 2290 lines - the largest and most complex file in the codebase

**Key Exports (`src/lib/index.ts`):**

- `superValidate`, `superForm` - Main APIs
- Proxy functions: `intProxy`, `numberProxy`, `dateProxy`, `fileProxy`, etc.
- Helpers: `defaults`, `defaultValues`, `setError`, `setMessage`, `withFiles`
- Types: `SuperValidated`, `Infer`, `InferIn`, `ValidationErrors`, `FormOptions`

### 3. **Data Flow**

```
Schema Definition (Zod/Valibot/etc.)
    ↓
Adapter wraps schema
    ↓
superValidate (server) → SuperValidated object
    ↓
Pass to client via PageData/ActionData
    ↓
superForm (client) → Reactive stores + enhance action
    ↓
Form submission → Server validation → Client update
```

### 4. **Key Subsystems**

**Traversal & Paths** (`src/lib/traversal.ts`, `src/lib/stringPath.ts`):

- Handle nested object paths like `user.addresses[0].city`
- `traversePath`, `setPaths`, `pathExists`, `comparePaths`
- `splitPath`, `mergePath` for string path manipulation

**Errors** (`src/lib/errors.ts`):

- `SuperFormError`, `SchemaError` custom errors
- `mapErrors`, `flattenErrors`, `updateErrors` for error transformation
- Merges schema errors with custom field errors

**Form Data** (`src/lib/formData.ts`):

- `parseRequest` - Extracts and coerces FormData to typed objects
- Handles arrays, files, nested structures
- Automatic type coercion (strings → numbers, dates, booleans)

**JSON Schema** (`src/lib/jsonSchema/`):

- Converts validation schemas to JSON Schema
- Generates HTML input constraints (min, max, pattern, required)
- `schemaDefaults` extracts default values from schemas
- `schemaShape` provides type information for nested structures

**Proxies** (`src/lib/client/proxies.ts`):

- Writable store wrappers that convert between form data and display formats
- `dateProxy` - Handles Date ↔ ISO string conversion
- `numberProxy`, `intProxy` - Number ↔ string with validation
- `fileProxy`, `filesProxy` - File input handling

**Progressive Enhancement** (`src/lib/client/form.ts`):

- `enhance` action (uses SvelteKit's `use:enhance`)
- Scrolls to first error, auto-focus, custom validity
- Works without JS, enhanced with JS

---

## Project Structure

```
src/
├── lib/
│   ├── adapters/           # Validation library adapters
│   │   ├── zod.ts          # Zod v3 adapter
│   │   ├── zod4.ts         # Zod v4 adapter
│   │   ├── valibot.ts, yup.ts, joi.ts, etc.
│   │   └── adapters.ts     # Core adapter types
│   ├── client/             # Client-side code (browser)
│   │   ├── superForm.ts    # Main client API (2290 lines)
│   │   ├── proxies.ts      # Data conversion proxies
│   │   ├── form.ts         # Progressive enhancement
│   │   ├── elements.ts     # DOM element helpers
│   │   ├── flash.ts        # Flash message handling
│   │   ├── customValidity.ts  # HTML5 validation API
│   │   ├── SuperDebug.svelte  # Debug component (Svelte 4)
│   │   └── SuperDebugRuned.svelte # Debug component (Svelte 5)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ciscoheat/sveltekit-superforms](https://github.com/ciscoheat/sveltekit-superforms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
