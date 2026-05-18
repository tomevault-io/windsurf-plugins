---
trigger: always_on
description: Bun + TypeScript monorepo with a CLI/TUI, Hono HTTP server, SolidJS web frontend, and native Rust FFI tools. Default branch is `dev`. Package manager: `bun@1.3.11`.
---

# IronCode Agent Guidelines

Bun + TypeScript monorepo with a CLI/TUI, Hono HTTP server, SolidJS web frontend, and native Rust FFI tools. Default branch is `dev`. Package manager: `bun@1.3.11`.

## Project layout

| Path                                                      | Description                                                                           |
| --------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `packages/ironcode`                                       | Core CLI, TUI (SolidJS + OpenTUI), Hono server, AI tools, session management          |
| `packages/ironcode/native/tool`                           | Rust crate (`ironcode-tool`) — grep, glob, edit, read, VCS, fuzzy match, etc. via FFI |
| `packages/sdk/js`                                         | Auto-generated TypeScript SDK from OpenAPI spec                                       |
| `packages/plugin`                                         | Public plugin API (`Plugin`, `ToolDefinition`, `Hooks`)                               |
| `packages/util`                                           | Shared utilities (`NamedError`, typed error factory)                                  |
| `packages/script`                                         | Internal build/release scripts                                                        |
| `packages/slack`, `packages/telegram`, `packages/discord` | Chat bot integrations                                                                 |

## Build / lint / test commands

```bash
# Install
bun install

# Dev servers
bun --cwd packages/ironcode dev              # CLI/TUI
bun --cwd packages/app run dev:web            # Web frontend

# Build
bun --cwd packages/<pkg> run build

# Typecheck (runs turbo across all packages — most use tsgo, not tsc)
bun run typecheck

# Format (Prettier — semi:false, printWidth:120, config is inline in root package.json)
./script/format.ts

# Regenerate SDK after changing server routes
./script/generate.ts
```

### Running tests

Tests use Bun's built-in test runner. **Never run tests from the repo root** — the root `test` script deliberately exits 1.

```bash
# All tests in a package
bun --cwd packages/ironcode test

# Single test file
bun --cwd packages/ironcode test test/tool/grep.test.ts

# Filter by name substring
bun --cwd packages/ironcode test --filter "basic search"

# Rust crate tests/benchmarks (run from crate directory)
cargo test      # workdir: packages/ironcode/native/tool
cargo bench
```

Test files live in `packages/ironcode/test/` organized by domain (tool/, session/, provider/, util/, config/, server/, etc.). All tests use `*.test.ts` — no `.spec.ts` convention.

### Pre-push hook (Husky)

The pre-push hook validates the Bun version matches `packageManager` in root `package.json`, then runs `bun typecheck`. No pre-commit hook.

## Code style & conventions

### Namespace pattern (dominant architecture)

Namespaces own their domain — types and runtime functions coexist:

```ts
export namespace Session {
  const log = Log.create({ service: "session" })

  export const Info = z.object({ id: z.string(), title: z.string() })
  export type Info = z.output<typeof Info>

  export const create = fn(CreateInput, async (input) => { ... })
}
```

Key namespaces: `Session`, `Config`, `Tool`, `Bus`, `BusEvent`, `Storage`, `Server`, `Agent`, `Log`, `Instance`.

### Imports

- Relative imports for local modules: `import { Log } from "../util/log"`
- Path aliases in `packages/ironcode`: `@/*` → `./src/*`, `@tui/*` → `./src/cli/cmd/tui/*`
- Named imports only for local code; default imports for third-party when appropriate (`import z from "zod"`)
- ESM throughout (`"type": "module"`)

### Naming

- Variables & functions: `camelCase`
- Types, interfaces, namespaces, enums: `PascalCase`
- Constants: `UPPER_SNAKE` only for true runtime constants
- DB columns (Drizzle): `snake_case`

### Formatting

- Prettier: `semi: false`, `printWidth: 120` (configured in root `package.json`, no `.prettierrc` file)
- 2-space indent, LF line endings, UTF-8
- No ESLint, Biome, or editorconfig — Prettier is the only formatter
- Run `./script/format.ts` before committing

### Types & validation

- Zod at every boundary: HTTP payloads, CLI args, config, tool inputs
- Dual-declaration pattern: `export const Info = z.object({...})` + `export type Info = z.output<typeof Info>`
- `fn()` helper wraps functions with Zod input validation
- `z.discriminatedUnion()` for union types; `.safeParse()` for non-throwing validation
- Avoid `any` — prefer `unknown` validated through Zod, explicit interfaces, or generics
- Use `const` by default; `let` only for deliberate mutability

### Error handling

- `NamedError.create("NotFoundError", z.object({...}))` from `@ironcode-ai/util/error` for typed errors
- `NamedError.Unknown` as generic catch-all
- Avoid throwing for control flow — use Result-like return values for internal APIs
- `try`/`catch` at IO boundaries; `.catch(() => {})` for fire-and-forget
- Convert external errors to typed shapes at boundaries before returning

### Logging & events


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KSD-CO/IronCode](https://github.com/KSD-CO/IronCode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
