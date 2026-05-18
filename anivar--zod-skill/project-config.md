---
trigger: always_on
description: > This document is for AI agents and LLMs to follow when writing, reviewing, or debugging Zod schemas. It compiles all rules and references into a single executable guide.
---

# Zod v4 — Complete Guide

> This document is for AI agents and LLMs to follow when writing, reviewing, or debugging Zod schemas. It compiles all rules and references into a single executable guide.

**Baseline:** zod ^4.0.0 with TypeScript ^5.5

---

## Abstract

Zod is a TypeScript-first schema validation library for runtime type checking at system boundaries — API input, form data, environment variables, external services. Use `z.infer<typeof Schema>` for type extraction, `safeParse()` for validation, and composition methods (`.pick()`, `.omit()`, `.partial()`) to derive schema variants. Zod v4 introduces breaking changes to string formats, enums, error handling, and recursive types. This guide also covers architectural placement (where to parse), schema organization, versioning strategy, and production observability.

---

## Table of Contents

1. [Parsing & Type Safety](#1-parsing--type-safety) — CRITICAL
2. [Schema Design](#2-schema-design) — CRITICAL
3. [Refinements & Transforms](#3-refinements--transforms) — HIGH
4. [Error Handling](#4-error-handling) — HIGH
5. [Performance & Composition](#5-performance--composition) — MEDIUM
6. [v4 Migration](#6-v4-migration) — MEDIUM
7. [Advanced Patterns](#7-advanced-patterns) — MEDIUM
8. [Architecture & Boundaries](#8-architecture--boundaries) — CRITICAL/HIGH
9. [Observability](#9-observability) — HIGH/MEDIUM

---

## 1. Parsing & Type Safety
**Impact: CRITICAL**

### Rule: Use safeParse() for User Input

`parse()` throws on invalid input. Use `safeParse()` which returns a discriminated result.

```typescript
// INCORRECT — try/catch is verbose and catches unrelated errors
try {
  const user = UserSchema.parse(data)
} catch (e) {
  if (e instanceof z.ZodError) { ... }
}

// CORRECT — discriminated union result
const result = UserSchema.safeParse(data)
if (!result.success) {
  console.log(result.error.issues)
} else {
  console.log(result.data)
}
```

Use `parse()` only for internal data that should never be invalid (config, constants).

### Rule: Use parseAsync for Async Refinements

When a schema has async `.refine()` or `.transform()`, sync `.parse()` throws. Must use `parseAsync()` or `safeParseAsync()`.

```typescript
const UniqueEmail = z.email().refine(
  async (email) => !(await db.users.exists({ email })),
  { error: "Email already registered" }
)

// INCORRECT — throws error with async refinement
const result = UniqueEmail.safeParse(input)

// CORRECT
const result = await UniqueEmail.safeParseAsync(input)
```

### Rule: Infer Types from Schemas

Never manually define TypeScript types alongside Zod schemas. Use `z.infer` and `z.input`.

```typescript
const UserSchema = z.object({
  name: z.string(),
  email: z.email(),
  age: z.number().min(0),
})

// Output type (after transforms)
type User = z.infer<typeof UserSchema>

// Input type (before transforms — useful for forms)
type UserInput = z.input<typeof UserSchema>
```

### Schema Types Quick Reference

| Type | Syntax |
|------|--------|
| String | `z.string()` |
| Number | `z.number()`, `z.int()`, `z.float()` |
| Boolean | `z.boolean()` |
| BigInt | `z.bigint()` |
| Date | `z.date()` |
| Literal | `z.literal("foo")`, `z.literal(42)` |
| Enum | `z.enum(["a", "b"])`, `z.enum(TSEnum)` |
| Email | `z.email()` |
| URL | `z.url()` |
| UUID | `z.uuid()` |
| String→Bool | `z.stringbool()` |
| ISO DateTime | `z.iso.datetime()` |
| File | `z.file()` |
| JSON | `z.json()` |
| Any | `z.any()` |
| Unknown | `z.unknown()` |
| Never | `z.never()` |

---

## 2. Schema Design
**Impact: CRITICAL**

### Rule: Handle Unknown Keys Explicitly

`z.object()` silently strips unknown keys. Choose the right variant:

```typescript
// z.object() — strips unknown keys (default)
const Safe = z.object({ host: z.string() })
Safe.parse({ host: "localhost", debug: true })
// { host: "localhost" } — debug is gone

// z.strictObject() — rejects unknown keys
const Strict = z.strictObject({ host: z.string() })
Strict.parse({ host: "localhost", debug: true })
// ZodError: unrecognized key "debug"

// z.looseObject() — preserves unknown keys
const Loose = z.looseObject({ host: z.string() })
Loose.parse({ host: "localhost", debug: true })
// { host: "localhost", debug: true }
```

| Variant | Unknown keys | Use when |
|---------|-------------|----------|
| `z.object()` | Strips | Sanitizing user input |
| `z.strictObject()` | Rejects | API contracts, config validation |
| `z.looseObject()` | Preserves | Proxying data, forwarding payloads |

### Rule: Use Discriminated Unions

For tagged object unions, use `z.discriminatedUnion()` instead of `z.union()`.

```typescript
// INCORRECT — sequential matching, poor error messages
const Shape = z.union([
  z.object({ type: z.literal("circle"), radius: z.number() }),
  z.object({ type: z.literal("square"), side: z.number() }),
])

// CORRECT — O(1) dispatch, targeted errors
const Shape = z.discriminatedUnion("type", [
  z.object({ type: z.literal("circle"), radius: z.number() }),
  z.object({ type: z.literal("square"), side: z.number() }),
])
```

### Rule: Coercion Pitfalls — Boolean Strings

`z.coerce.boolean()` uses JavaScript `Boolean()` — `Boolean("false")` is `true`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anivar/zod-skill](https://github.com/anivar/zod-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
