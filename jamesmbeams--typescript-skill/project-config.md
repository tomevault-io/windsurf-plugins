---
trigger: always_on
description: >
---


# TypeScript Best Practices — Matt Pocock Style

A comprehensive reference for writing high-quality TypeScript, compiled from Matt Pocock's publicly available teachings at [Total TypeScript](https://www.totaltypescript.com). Not affiliated with or endorsed by Matt Pocock or Total TypeScript.

---

## 1. `type` vs `interface`

**Default to `type`. Use `interface` only for object inheritance.**

### Why `type` by default:
- `type` can express unions, mapped types, conditional types — `interface` cannot
- Interfaces with the same name in the same scope **silently merge** (declaration merging), causing unexpected bugs
- Interfaces lack an implicit index signature, causing friction with `Record` types

### When to use `interface`:
- When you need `extends` for object inheritance — it's faster than `&` intersections
- TypeScript caches interfaces by name internally; intersection types are recomputed

```typescript
// ✅ Default: use type
type User = {
  id: string;
  name: string;
};

type StringOrNumber = string | number;

// ✅ Use interface for object inheritance (faster than &)
interface WithId {
  id: string;
}

interface User extends WithId {
  name: string;
}

// ❌ Avoid: intersection for inheritance (slower)
type User = WithId & {
  name: string;
};

// ❌ Danger: declaration merging surprise
interface Config {
  debug: boolean;
}
interface Config {  // silently merges!
  verbose: boolean;
}
```

---

## 2. Avoid Enums — Use Const Objects or Union Types

Enums are one of TypeScript's few non-type-level features. They emit JavaScript, have surprising behaviors, and are largely unnecessary.

### Problems with enums:
- Numeric enums allow unsafe reverse lookups
- They emit extra JavaScript (an IIFE)
- They can't be used with `satisfies`
- `const enum` has its own issues (inlining across module boundaries)

```typescript
// ❌ Avoid: enum
enum Status {
  Active = "active",
  Inactive = "inactive",
}

// ✅ Option 1: Union type (simplest)
type Status = "active" | "inactive";

// ✅ Option 2: Const object (when you need runtime values + type)
const Status = {
  Active: "active",
  Inactive: "inactive",
} as const;

type Status = (typeof Status)[keyof typeof Status];
// Result: "active" | "inactive"

// Usage — same as an enum but no emitted code surprises
function setStatus(status: Status) {}
setStatus(Status.Active); // works
setStatus("active");      // also works
```

**Rule:** For simple sets of strings, use a union type. When you need a named object for runtime access, use `as const` object + derived type.

---

## 3. The `satisfies` Operator

`satisfies` validates that a value conforms to a type **without widening it**. The value beats the type.

### When to use each:
- **Colon annotation (`:`)** — type beats value. Use for explicit typing, function parameters, variables that will be reassigned
- **`satisfies`** — value beats type. Use when you want validation AND narrow inference
- **`as`** — almost never. It lets you lie to TypeScript
- **No annotation** — let TypeScript infer. This is the default and often the best choice

```typescript
// ❌ Colon widens — loses key information
const routes: Record<string, {}> = {
  "/": {},
  "/users": {},
};
routes.anything; // no error — type is too wide

// ✅ satisfies: validates AND preserves narrow type
const routes = {
  "/": {},
  "/users": {},
} satisfies Record<string, {}>;

routes["/"];       // ✅ works
routes.anything;   // ❌ error — keys are known

// ✅ satisfies for config objects
type ColorConfig = Record<string, string | string[]>;

const palette = {
  red: "#ff0000",
  green: ["#00ff00", "#00ee00"],
} satisfies ColorConfig;

// TypeScript knows palette.red is string, palette.green is string[]
palette.green.map(g => g.toUpperCase()); // ✅ no assertion needed
```

### `as const satisfies` — the power combo

Combining `as const` with `satisfies` gives you **literal inference + type validation** in one expression. This is one of the most useful patterns in modern TypeScript.

```typescript
// ✅ as const satisfies — validates shape AND preserves literals
const routes = {
  home: "/",
  users: "/users",
  admin: "/admin",
} as const satisfies Record<string, string>;

// TypeScript knows: routes.home is "/", not string
// AND it validated that all values are strings

// ✅ Great for config with known shapes
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
  createUser: { method: "POST", path: "/users" },
} as const satisfies Record<string, { method: string; path: string }>;

// endpoints.getUser.method is "GET", not string
// endpoints.createUser.path is "/users", not string

// ❌ Without as const — literals are widened
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
} satisfies Record<string, { method: string; path: string }>;
// endpoints.getUser.method is string — literals lost

// ❌ Without satisfies — no shape validation
const endpoints = {
  getUser: { method: "GET", path: "/users/:id" },
} as const;
// Typos or missing fields won't be caught
```

**Rule:** Use `as const satisfies` when you want to lock down a value to literal types while also validating it conforms to a shape. Use `satisfies` alone when you don't need literal narrowing. Use `:` when you explicitly want the wider type.

---

## 4. TSConfig Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jamesmbeams/typescript-skill](https://github.com/jamesmbeams/typescript-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
