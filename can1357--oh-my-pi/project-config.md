---
trigger: always_on
description: This repo contains multiple packages, but **`packages/coding-agent/`** is the primary focus. Unless otherwise specified, assume work refers to this package.
---

# Development Rules

## Default Context

This repo contains multiple packages, but **`packages/coding-agent/`** is the primary focus. Unless otherwise specified, assume work refers to this package.

**Terminology**: When the user says "agent" or asks "why is agent doing X", they mean the **coding-agent package implementation**, not you (the assistant). The coding-agent is a CLI tool that uses Claude—questions about its behavior refer to the code in `packages/coding-agent/`, not your current session.

### Package Structure

| Package                 | Description                                          |
| ----------------------- | ---------------------------------------------------- |
| `packages/ai`           | Multi-provider LLM client with streaming support     |
| `packages/agent`        | Agent runtime with tool calling and state management |
| `packages/coding-agent` | Main CLI application (primary focus)                 |
| `packages/tui`          | Terminal UI library with differential rendering      |
| `packages/natives`      | bindings for native text/image/grep operations       |
| `packages/stats`        | Local observability dashboard (`omp stats`)          |
| `packages/utils`        | Shared utilities (logger, streams, temp files)       |
| `crates/pi-natives`     | Rust crate for performance-critical text/grep ops    |

## Code Quality

- No `any` types unless absolutely necessary
- Prefer `export * from "./module"` over named re-export-from blocks, including `export type { ... } from`. In pure `index.ts` barrel files (re-exports only), use star re-exports even for single-specifier cases. If star re-exports create symbol ambiguity, remove the redundant export path instead of keeping duplicate exports.
- **No `private`/`protected`/`public` keyword on class fields or methods** — use ES native `#` private fields for encapsulation; leave members that need external access as bare (no keyword). The only place `private`/`protected`/`public` is allowed is on **constructor parameter properties** (e.g., `constructor(private readonly session: ToolSession)`), where TypeScript requires the keyword for the implicit field declaration.

  ```typescript
  // BAD: TypeScript keyword privacy
  class Foo {
      private bar: string;
      private _baz = 0;
      protected qux(): void { ... }
      public greet(): void { ... }
  }

  // GOOD: ES native # for private, bare for accessible
  class Foo {
      #bar: string;
      #baz = 0;
      qux(): void { ... }
      greet(): void { ... }
  }

  // OK: constructor parameter properties keep the keyword
  class Service {
      constructor(private readonly session: ToolSession) {}
  }
  ```

- **NEVER use `ReturnType<>`** — it obscures types behind indirection. Use the actual type name instead. Look up return types in source or `node_modules` type definitions and reference them directly.

  ```typescript
  // BAD: Indirection through ReturnType
  let timer: ReturnType<typeof setTimeout> | null = null;
  let stmt: ReturnType<Database["prepare"]>;
  let stat: Awaited<ReturnType<typeof fs.stat>>;

  // GOOD: Use the actual type
  let timer?: NodeJS.Timeout;
  let stmt: Statement;
  let stat: Stats;
  ```

  If a function's return type has no exported name, define a named type alias at the call site — don't use `ReturnType<>`.

- Check node_modules for external API type definitions instead of guessing
- **NEVER use inline imports** - no `await import("./foo.js")`, no `import("pkg").Type` in type positions, no dynamic imports for types. Always use standard top-level imports.
- NEVER remove or downgrade code to fix type errors from outdated dependencies; upgrade the dependency instead
- Always ask before removing functionality or code that appears to be intentional
- **NEVER build prompts in code** — no inline strings, no template literals, no string concatenation. Prompts live in static `.md` files; use Handlebars for any dynamic content.
- **Import static text files via Bun** — use `import content from "./prompt.md" with { type: "text" }` instead of `readFileSync`
- **Use `Promise.withResolvers()`** instead of `new Promise((resolve, reject) => ...)` — cleaner, avoids callback nesting, and the resolver functions are properly typed:

  ```typescript
  // BAD: Verbose, callback nesting
  const promise = new Promise<string>((resolve, reject) => { ... });

  // GOOD: Clean destructuring, typed resolvers
  const { promise, resolve, reject } = Promise.withResolvers<string>();
  ```

## Bun Over Node

This project uses Bun. Use Bun APIs where they provide a cleaner alternative; use `node:fs` for operations Bun doesn't cover.

**NEVER spawn shell commands for operations that have proper APIs** (e.g., `Bun.spawnSync(["mkdir", "-p", dir])` — use `mkdirSync` instead).

### Process Execution

**Prefer Bun Shell** (`$` template literals) for simple commands:

```typescript
import { $ } from "bun";

// Capture output
const result = await $`git status`.cwd(dir).quiet().nothrow();
if (result.exitCode === 0) {
	const text = result.text();
}

// Fire and forget
$`do-stuff ${tmpFile}`.quiet().nothrow();
```

**Use `Bun.spawn`/`Bun.spawnSync`** only when:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [can1357/oh-my-pi](https://github.com/can1357/oh-my-pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
