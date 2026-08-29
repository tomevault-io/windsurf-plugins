---
trigger: always_on
description: - Applies to the entire repository.
---

Basango — AGENTS.md

Scope
- Applies to the entire repository.
- Use these conventions when adding/modifying code, scripts, or docs.

Environment
- Node: >= 22
- Package manager: Bun `1.3.x`
- Task runner: Turborepo
- Lint/format: Biome

Workspace Layout
- `workspaces`: `apps/*` and `packages/*`.
- Internal packages use the `@basango/` scope and `workspace:*` versions.
- Avoid nested packages like `apps/**` or `packages/**`.

Documentation
- Engineering documentation belongs under `docs/`.
- Before changing a TypeScript application or shared package, read `docs/web/README.md` and the relevant package or feature documentation.
- All authored TypeScript and React follows `docs/web/code-style.md`. It is authoritative for naming, readability, React patterns, module boundaries, and package interfaces.
- Update architecture documentation when a change introduces a new application, package, process boundary, public package entrypoint, or workspace dependency direction.

Packages
- `@basango/logger`: Pino wrapper. Prefer named import `import { logger } from "@basango/logger"`.
- `@basango/db`: Drizzle ORM for Postgres. Import via defined subpaths (`./client`, `./queries`, `./schema`, `./utils`).
- `@basango/ui`: Shared UI.
- `@basango/tsconfig`: Shared TS configs. Extend this in apps/packages.

Conventions
- ESM-only: set `"type": "module"` for packages that ship code.
- TypeScript everywhere. Use `extends: "@basango/tsconfig/base.json"` when possible.
- Prefer named exports in libraries. Avoid barrel files unless necessary.
- Use `workspace:*` for internal dependencies; do not hardcode versions.
- Keep changes minimal and localized; avoid cross-cutting refactors without discussion.
- When using tRPC in React, always compose `useQuery`/`useMutation` from TanStack with `trpc.*.queryOptions`/`mutationOptions` instead of calling `trpc.*.useQuery`/`useMutation` helpers directly (they are deprecated).

Architecture
- Organize application code by bounded context and product capability before technical role.
- Keep route composition, navigation, environment access, and platform behavior in the owning application.
- Start new behavior in its owning application. Extract it only for at least two real consumers or a clear package-owned responsibility.
- Packages expose small, intentional interfaces and hide cohesive implementation details.
- No package may import from an application. No client application may import `@basango/db` or `@basango/logger`.
- Dashboard code imports API router types only through the explicit `@basango/api/trpc/routers/_app` export.
- Mobile may share platform-neutral domain contracts but must not import the DOM-based `@basango/ui` package.
- Follow the dependency graph in `docs/web/README.md`. A new lateral package dependency requires a real ownership relationship and a documentation update.

TypeScript
- Use `type`. Use `interface` only when declaration merging or third-party module augmentation requires it.
- Define API transport, persisted structured data, and structured form contracts as Zod schemas.
- Infer TypeScript types from Zod rather than duplicating contract types manually. A focused typed parser is sufficient for one simple route or environment value.
- Use camelCase for schema properties.
- Use function declarations for named functions, components, hooks, handlers, formatters, predicates, and factories. Reserve arrow functions for inline callbacks and expression-based wrappers.
- Use `T[]` and `readonly T[]`, `unknown` at untrusted seams, and discriminated unions instead of enums.
- Use `import type` for type-only dependencies.
- Prefer `undefined` for omitted internal values. Preserve `null` only where a transport or persisted contract distinguishes it.

Readability
- Separate logical blocks with one blank line: declaration groups, control flow, side effects, render guards, and final returns must read as distinct semantic paragraphs.
- Put one blank line between every top-level schema, type, constant, function, and component declaration.
- Always use braces and multiline bodies for `if`, `for`, `while`, `switch`, and `try`; never compress a guard or side effect onto one line.
- Inside a function, closely related side-effect-free declarations may stay together. Do not add padding immediately inside braces or between `if`/`else` and `try`/`catch`.

Imports and Exports
- Use relative imports inside one feature slice or package.
- Use `#dashboard/*` or `#mobile/*` when crossing an application seam.
- Use `@basango/*` only when crossing a declared package seam. Never import a package from itself through its public alias.
- Never import another workspace's undeclared source or internal path. Do not use TypeScript path mappings as a substitute for package exports.
- Omit `.ts` and `.tsx` extensions. Use named React imports rather than `import * as React` in authored code.
- Prefer named exports. Default exports are limited to framework requirements.
- Keep `index.ts` interface- or composition-only and use explicit exports instead of `export *`.
- Do not pass through another package's domain symbols. Consumers import a symbol from its owner.
- Public package paths must be intentional `package.json` exports. Wildcard exports are limited to the UI registry's component-per-file convention.

React

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bernard-ng/basango](https://github.com/bernard-ng/basango) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
