---
trigger: always_on
description: Breaking changes are accepted at major-version boundaries. Add a `changeset` for every change to `packages/envapt/**`.
---

# Envapt Agent Guidelines

Breaking changes are accepted at major-version boundaries. Add a `changeset` for every change to `packages/envapt/**`.

---

## Repository Policy

**Zero Technical Debt.** No workarounds, hacks, or temporary compatibility layers. Choose the cleanest architecture; break things if needed to get it right.

**Scope discipline.** Only implement what was explicitly asked. Surface additions as a question before implementing — never silently add config tweaks, optimizations, feature flags, or abstractions not in the task description.

**No dead code.** No commented-out code, half-finished `// TODO: complete later` implementations, or unused exports. Before adding `export` to a symbol, verify it is consumed outside the file. If it isn't, drop the `export`. If nothing uses it, delete it.

**Do not edit `AGENTS.md`, `CLAUDE.md` (symlink), or any file in `.changeset/` without explicit permission.**

**Respect `Note for Agent:` comments.** The user may add these mid-flight as deliberately-failing type errors or lint errors so they surface when you run checks. Read and honor them before continuing; remove the comment when done.

```ts
// This will cause a lint error
('Note for Agent: switch to the new effects API');

type NoteForAgentAddedByTheUser = 'switch to the new effects API';
const x: NoteForAgentAddedByTheUser = 42; // forces a type error
```

---

## Design Patterns

- **OOP for complex domain logic** (inheritance & composition). **Plain functions for small, stateless utilities.** envapt's public surface leans on classes (`Envapter` and the mixin chain in `core/`); extend or compose those rather than re-implementing parallel function pipelines.

```ts
// Bad
export function getNumber() {}
export function getBoolean() {}

// Good — already the pattern in `core/PrimitiveMethods.ts`
export class PrimitiveMethods extends EnvironmentMethods {
    static getNumber() {}
    static getBoolean() {}
}
```

- **No static-only classes as namespaces.** Use named exports instead. The exception in envapt is `BuiltInConverters` which is a dispatch-table helper — that's a real OOP shape for the lookup pattern, not a namespace.

```ts
// Bad
export class Utils {
    static foo() {}
}

// Good
export function foo() {}
```

- **Function declarations for complex exported functions.** Arrow expressions for inline callbacks and short utilities only — no block-bodied exported arrows.

```ts
// Bad
export const compute = () => {
    /* large */
};

// Good
export function compute() {
    /* large */
}
```

- **DRY and SOLID.** No premature abstractions — three similar lines is better than a wrong abstraction. Wait for the fourth use before extracting.

- **YAGNI.** Don't add features, config, abstractions, or infrastructure for hypothetical future requirements. Ship what the task requires; surface everything else as a question first.

- **No premature optimization.** envapt is config-loading code — not hot-path. Readable, correct first. Profile before optimizing.

- **Split large files** (~200+ lines or multiple unrelated responsibilities) into focused modules. The `core/` mixin chain (`EnvapterBase` → `EnvironmentMethods` → `PrimitiveMethods` → `AdvancedMethods` → `Envapter`) is the canonical example — each file has one responsibility.

---

## Type Standards

- **No `any` in production code.** Use `unknown` then narrow with a type guard. If a third-party library forces a cast, prefer a single `as Expected` with `// justified: <reason>`.
- **No `as unknown as T` double casts.** Fix the declaration, write a type guard, or refactor the API.
- **Don't cast values that are already correctly typed** — adjust the type instead.
- **Prefer `?.` and `??`** for genuinely optional branches — not to suppress errors or hide broken assumptions. See `.github/skills/code-quality/FAIL-FAST-RULES.md` for when NOT to reach for them.
- **Prefer `import type { T } from 'pkg'`** for type-only imports. Avoid inline `import('pkg').T`.
- **Use `type-fest` utility types** if you need structural transforms beyond what's already in `types/`. envapt's `types/` modules re-export project-specific aliases — check there first.
- **Tests may use pragmatic fixture casts** (`as unknown as Test`) — always include a short justification comment. Tests must not use `as any`.
- **To disable an ESLint rule inline:** `// eslint-disable-next-line <rule> -- <reason>`. Never file-wide or project-wide.

```ts
// Bad
let v: any;
const a = (obj as any).x ?? 'd';
const v = x as unknown as T;

// Good
let v: unknown; // then narrow with a type guard
const a = obj?.x ?? 'd';
if (isT(x)) {
    const v = x;
}
import type { Foo } from 'pkg'; // not import('pkg').Foo
```

---

## Imports & Dependencies

- **No cross-package source paths.** envapt has one publishable package today, but if a second one is ever added: no `paths` or `include` reaching `../../packages/x/src`. Consume via package exports only — the `exports` map in each package's `package.json` is authoritative.
- **Use path aliases** if any get added; otherwise relative imports are fine for this codebase size.
- **Add deps with `pnpm add --filter envapt`.** Inspect type declarations before relying on them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [materwelonDhruv/envapt](https://github.com/materwelonDhruv/envapt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
