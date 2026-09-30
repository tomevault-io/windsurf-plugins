---
trigger: always_on
description: **Safety, performance, and developer experience — in that order.** Every rule in
---

# AGENTS.md — TypeScript Standards

## Design goals

**Safety, performance, and developer experience — in that order.** Every rule in
this document exists to serve one of those goals; style is not taste. These rules
make invalid states **unrepresentable**, move failures to compile time or to
system boundaries, and make the failures that remain **loud**. They are
language-canonical: no framework, runtime, or platform assumptions. Follow them
when writing or editing code in this repository.

We operate a **zero-technical-debt** policy: do it right the first time, because
the second time may not transpire, and a problem solved in design is many times
cheaper than one solved in production. When a rule conflicts with existing code,
prefer the rule and refactor the old code when touched.

**Always say why.** When a rule, a decision, or a piece of code needs a
rationale, write the rationale — in this document, in comments, in commit
messages. A stated reason lets the reader evaluate the decision, not just obey
it.

## Non-negotiables

1. **No `any`.** Use `unknown` and narrow.
2. **No type assertions (`as T`, `<T>x`) on data you don't fully control.**
   Assertions are permitted only inside a validated constructor or type guard.
3. **No `enum`.** Use string-literal unions + `as const`.
4. **No `Partial<T>` on function inputs.** Use explicit `Pick<>` types.
5. **Every value crossing a boundary (network, file, env, user input,
   `JSON.parse`) is parsed and validated before use** — never cast and trust.
6. **Immutability by default**: `readonly` properties, `ReadonlyArray<T>`,
   `as const`. Mutation must be local, centralized, and justified (§11).
7. **Functions are total where possible**: every declared input type produces a
   declared output — absence and failure are part of the return type.
8. **Two error channels, never mixed**: expected operating failures are values
   (`Result`); programmer errors are assertions that crash (§9).
9. **Put a limit on everything.** Every loop, queue, retry, and collection has
   a stated upper bound (§10).
10. **Assert invariants.** Preconditions, postconditions, and invariants are
    checked at runtime; core logic averages at least two assertions per
    function (§9).

## tsconfig (required flags)

```jsonc
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "verbatimModuleSyntax": true
  }
}
```

## ESLint (required rules)

```jsonc
{
  "rules": {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unsafe-assignment": "error",
    "@typescript-eslint/no-unsafe-member-access": "error",
    "@typescript-eslint/no-unsafe-call": "error",
    "@typescript-eslint/no-unsafe-return": "error",
    "@typescript-eslint/consistent-type-assertions": ["error", { "assertionStyle": "never" }],
    "@typescript-eslint/prefer-readonly": "error",
    "@typescript-eslint/switch-exhaustiveness-check": "error",
    "curly": ["error", "all"],
    "max-lines-per-function": ["error", { "max": 70, "skipBlankLines": true, "skipComments": true }]
  }
}
```

Formatting: Prettier with `printWidth: 100`. **100 columns is a hard limit,
without exception** — nothing may hide behind a horizontal scrollbar. Use the
full width; never go beyond.

---

## Core patterns

### 1. Discriminated unions + exhaustive handling

Model state as a union where data exists **only in the states that have it** —
never parallel optional fields and booleans.

```ts
export type LoadState<T> =
  | { kind: "idle" }
  | { kind: "loading" }
  | { kind: "error"; message: string }
  | { kind: "ready"; data: T };

export function renderState<T>(s: LoadState<T>, render: (d: T) => string): string {
  switch (s.kind) {
    case "idle":    return "";
    case "loading": return "…";
    case "error":   return s.message;   // Only exists in this variant.
    case "ready":   return render(s.data);
    default: {
      const _exhaustive: never = s;     // Adding a variant = compile error here.
      return _exhaustive;
    }
  }
}
```

Prefer this over `enum` — unions narrow properly, erase at runtime, and
interoperate with plain data:

```ts
export type Status = "draft" | "pending" | "approved" | "rejected";
```

### 2. Parse, don't validate — boundaries

A value is checked **once** at the boundary; after that, its type proves
validity. Hand-rolled guards are the canonical form (see §17 before reaching
for a schema library — one is justified only for genuinely large shapes):

```ts
export interface Project {
  readonly id: number;
  readonly title: string;
  readonly status: Status;
}

const STATUSES = ["draft", "pending", "approved", "rejected"] as const;

export function parseProject(input: unknown): Project {
  if (typeof input !== "object" || input === null) {
    throw new Error("Project: expected object");
  }
  const p = input as Record<string, unknown>;
  if (!Number.isSafeInteger(p.id)) throw new Error("Project.id: expected safe integer");
  if (typeof p.title !== "string" || p.title.length === 0) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flowtricks/stacki](https://github.com/flowtricks/stacki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
