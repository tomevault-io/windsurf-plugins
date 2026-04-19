---
trigger: always_on
description: **Expect** is a CLI tool that lets coding agents (Claude Code, Codex CLI, Cursor) automatically test code changes in a real browser. The workflow:
---

# Expect

## Project Overview

**Expect** is a CLI tool that lets coding agents (Claude Code, Codex CLI, Cursor) automatically test code changes in a real browser. The workflow:

1. User runs `expect` in their terminal
2. The tool scans unstaged git changes or a branch diff
3. An AI agent generates a test plan describing how to validate the changes
4. The user reviews and approves the plan in an interactive TUI
5. The agent executes the test plan against a live browser instance
6. Results are displayed with pass/fail status and session recordings

**Target:** Developers using terminal coding agents who want automated browser-based validation of their code changes.

**Core architecture:**

- **pnpm monorepo** — `expect-cli` (Ink TUI), `@expect/supervisor` (orchestration), `@expect/agent` (LLM backend), `@expect/browser` (Playwright automation), `@expect/cookies` (browser profile extraction), `@expect/shared` (domain models)
- **Supervisor** — Owns all state management, agent lifecycle, and git operations. The CLI is a stateless renderer of supervisor state.
- **Browser automation** — Playwright-based with MCP protocol support and rrweb session recording.
- **Cookie extraction** — Reads browser profile databases (Chrome, Firefox, Safari) so tests run with real authentication state.

**Tech stack:** Effect-TS, React + Ink (terminal UI), Playwright, TypeScript

See `.specs/` for implementation specs.

## Verify changes

```bash
pnpm check
```

## Code style

- `interface` over `type`. `Boolean` over `!!`. Arrow functions only.
- No comments unless it's a hack (`// HACK: reason`).
- No type casts (`as`) unless unavoidable.
- No unused code, no duplication.
- Descriptive variable names (no shorthands or 1-2 char names).
- kebab-case filenames.
- Magic numbers go in `constants.ts` as `SCREAMING_SNAKE_CASE` with unit suffixes (`_MS`, `_PX`).
- One focused utility per file in `utils/`.
- Namespace imports for Node built-ins `fs`, `os`, and `path` (e.g. `import * as fs from "node:fs"`, `import * as path from "node:path"`), not default or named imports.

## No Barrel Files

**Never create index.ts files that just re-export things.** The name `index.ts` lacks semantic meaning and adds unnecessary indirection. Import directly from the source file instead.

```ts
// GOOD - import directly from the source
import { Cookies } from "@expect/cookies/cookies";
import { TestPlan } from "@expect/shared/test-plan";

// BAD - don't create or use barrel files
// src/cookies/index.ts that just does: export * from "./cookies"
import { Cookies } from "@expect/cookies";
```

# Writing Effect Guide

**Code Examples**: For real-world Effect code patterns and implementation examples, search
the `.repos/effect` folder. This contains the Effect source code and is an excellent
reference for understanding how Effect APIs are used in practice. Use Grep/Glob to find
relevant snippets when you need to see how a specific function or pattern is implemented.

**Effect Atom**: For React integration patterns using Effect, refer to `.repos/effect/packages/atom/react` and `.repos/effect/packages/effect/src/unstable/reactivity/Atom.ts`.
This library provides reactive state management for React using Effect, with atoms and
hooks for building UI components that integrate cleanly with Effect services.

# Effect Rules

Effect v4 patterns for this codebase.

## Services — Use `ServiceMap.Service`

Never use `Effect.Service` or `Context.Tag`. Use `ServiceMap.Service` with `make:` property and explicit `static layer`.

```ts
import { Effect, Layer, ServiceMap } from "effect";

export class Cookies extends ServiceMap.Service<Cookies>()("@cookies/Cookies", {
  make: Effect.gen(function* () {
    const cdpClient = yield* CdpClient;

    const extract = Effect.fn("Cookies.extract")(function* (options: ExtractOptions) {
      yield* Effect.annotateCurrentSpan({ url: options.url });
      // ...
      return cookies;
    });

    return { extract } as const;
  }),
}) {
  static layer = Layer.effect(this)(this.make).pipe(Layer.provide(CdpClient.layer));
}
```

Key differences from `Effect.Service`:

- `make:` not `effect:`
- No `dependencies:` array — use `Layer.provide()` chaining on `static layer`
- No `accessors: true`
- `Layer.effect(this)(this.make)` for the layer

## Errors — Use `Schema.ErrorClass`

Use `Schema.ErrorClass` with explicit `_tag: Schema.tag(...)`. Define `message` as a class field derived from data, never as a schema field.

```ts
import { Schema } from "effect";

export class CookieDatabaseNotFoundError extends Schema.ErrorClass<CookieDatabaseNotFoundError>(
  "CookieDatabaseNotFoundError",
)({
  _tag: Schema.tag("CookieDatabaseNotFoundError"),
  browser: Schema.String,
}) {
  message = `Cookie database not found for ${this.browser}`;
}
```

Failing with an error — `.asEffect()` is not needed when using `return yield*`:

```ts
if (!databasePath) {
  return yield * new CookieDatabaseNotFoundError({ browser });
}
```

## Error Handling

Use `catchTag` / `catchTags` for specific errors. Never `catchAll` or `mapError`. Never `Effect.catch(...)` — always `Effect.catchTag("SpecificError", ...)`.

Infrastructure errors become defects:

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [millionco/expect](https://github.com/millionco/expect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
