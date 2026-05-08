---
trigger: always_on
description: - No `any` types unless absolutely necessary
---

# Development Rules

## Code Quality

- No `any` types unless absolutely necessary
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

## WASM Bindgen Resources

**Use `using` declarations** for wasm-bindgen objects — they implement `Symbol.dispose`:

```typescript
// BAD: Manual try/finally with .free()
const image = PhotonImage.new_from_byteslice(bytes);
try {
	return image.get_bytes();
} finally {
	image.free();
}

// GOOD: Automatic cleanup via Symbol.dispose
using image = PhotonImage.new_from_byteslice(bytes);
return image.get_bytes();
```

This applies to all wasm-bindgen generated classes (`CompiledPattern`, `PhotonImage`, etc.).

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

- Long-running processes (LSP servers, Python kernels)
Logs go to `~/.oms/logs/oms-*.log` with automatic rotation.
- Process control needed (signals, kill, complex lifecycle)

**Bun Shell methods:**

- `.quiet()` - suppress output (stdout/stderr to null)
- `.nothrow()` - don't throw on non-zero exit
- `.text()` - get stdout as string
- `.cwd(path)` - set working directory

### Sleep

**Prefer** `await Bun.sleep(ms)`  
**Avoid** `new Promise((resolve) => setTimeout(resolve, ms))`

### Node Module Imports

**NEVER use named imports from `node:fs` or `node:path`** — always use namespace imports:

```typescript
// BAD: Named imports
import { readdir, stat } from "node:fs/promises";
import { join, resolve } from "node:path";
import { tmpdir } from "node:os";

// GOOD: Namespace imports
import * as fs from "node:fs/promises";
import * as path from "node:path";
import * as os from "node:os";

// Then use: fs.readdir(), path.join(), etc.
```

**Choosing between `node:fs` and `node:fs/promises`:**

- **Async-only file** → `import * as fs from "node:fs/promises"`
- **Needs both sync and async** → `import * as fs from "node:fs"`, use `fs.promises.xxx` for async

### File I/O

**Prefer Bun file APIs:**

```typescript
// Read
const text = await Bun.file(path).text();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeprecatedLuke/oh-my-singularity](https://github.com/DeprecatedLuke/oh-my-singularity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
