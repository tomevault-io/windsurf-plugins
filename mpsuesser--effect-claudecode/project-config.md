---
trigger: always_on
description: Effect-first library for writing Claude Code hooks, plugins, skills, subagents, commands, and settings in a maximally Effect-native way. Wraps Claude Code's extensibility primitives (stdio hook processes, settings.json, plugin manifests, frontmatter files, .mcp.json) in Effect v4 idioms.
---

# AGENTS.md — effect-claudecode

Effect-first library for writing Claude Code hooks, plugins, skills, subagents, commands, and settings in a maximally Effect-native way. Wraps Claude Code's extensibility primitives (stdio hook processes, settings.json, plugin manifests, frontmatter files, .mcp.json) in Effect v4 idioms.

## Build / Lint / Test Commands

```sh
bun run test           # vitest run (all tests)
bun run typecheck      # tsc --noEmit
```

### Running a single test

```sh
bunx vitest run test/Hook/Events/PreToolUse.test.ts     # single file
bunx vitest run -t "denies rm -rf"                        # by test name pattern
bunx vitest run test/Hook/Runner.test.ts -t "schema decode"
```

### Verification before submitting

```sh
bun run test && bun run typecheck
```

## Project Structure

```
src/
  index.ts                  Barrel: re-exports Hook, Settings, Plugin, Frontmatter,
                            Mcp, Testing namespaces + cross-module errors.
  Hook.ts                   Re-export hub for all Hook/* modules.
  Hook/
    Runner.ts               Hook.runMain + Hook.dispatch. Stdio FFI boundary.
    Context.ts              HookContext namespace (ServiceMap.Service + accessors).
    Envelope.ts             Base HookEnvelope schema (session_id, cwd, etc.).
    Matcher.ts              Tool-name regex helpers for Pre/PostToolUse.
    Transcript.ts           FileSystem-backed transcript reader.
    Events/
      index.ts              Builds HookInput/HookOutput tagged unions.
      <EventName>.ts        One file per event — Input, Output, decision
                            helpers, and define(). 27 event files total.
  Settings.ts               Re-export hub for Settings/*.
  Settings/                 SettingsFile schema + Settings.load (user/project/local merge).
  Plugin.ts                 Re-export hub for Plugin/*.
  Plugin/                   plugin.json manifest schema + Plugin.define + Plugin.write.
  Frontmatter.ts            Re-export hub for Frontmatter/*.
  Frontmatter/              YAML-frontmatter parsers for skills, subagents, commands.
  Mcp.ts                    Re-export hub for Mcp/*.
  Mcp/                      .mcp.json schema (stdio/http/sse discriminated).
  Errors.ts                 All Schema.TaggedErrorClass declarations.
  Testing.ts                Test helpers: runHookWithMockStdin, fixtures,
                            expect*Decision assertions.

test/                       One file per source module.
vitest.config.ts            vitest config (plain vitest, no vite-plus in Phase 1).
vitest.setup.ts             @effect/vitest equality testers.
tsconfig.json               Strict TS with @effect/language-service plugin.
```

Single-package project. Bun is the package manager (`bun@1.3.11`). No monorepo tooling. See `/Users/m/.claude/plans/mighty-hatching-stallman.md` for the phased implementation plan.

## Code Style

### Formatting (manual conventions — no auto-formatter wired in Phase 1)

- Tabs for indentation (width 4), print width 80
- Single quotes, semicolons, no trailing commas
- Arrow parens always: `(x) => ...`
- LF line endings
- JSON files: 2-space indentation

### Imports

1. **Type-only imports first**: `import type { ... } from '...'`
2. **External namespace imports**: `import * as Effect from 'effect/Effect'`
3. **Internal imports last**: `import * as Envelope from './Envelope.ts'`

Canonical Effect aliases:

```ts
import * as Arr from 'effect/Array';
import * as Effect from 'effect/Effect';
import * as Layer from 'effect/Layer';
import * as Option from 'effect/Option';
import * as Schema from 'effect/Schema';
import * as ServiceMap from 'effect/ServiceMap';
import * as Stream from 'effect/Stream';
import * as Stdio from 'effect/Stdio';
```

Core combinators from root: `import { Effect, pipe, flow, Match } from 'effect'` — or from the dedicated module: `import * as Effect from 'effect/Effect'`.

**All local imports include `.ts` extension**: `'./Envelope.ts'`, not `'./Envelope'`.

### File & Naming Conventions

- **Source files**: PascalCase (`Runner.ts`, `Context.ts`, `PreToolUse.ts`)
- **Test files**: `<ModuleName>.test.ts` mirroring the source path under `test/`
- **Functions/variables**: camelCase (`runMain`, `fromEnvelope`, `matchTool`)
- **Types/interfaces/classes**: PascalCase (`HookDefinition`, `HookEnvelope`)
- **Service keys**: namespaced strings (`'effect-claudecode/HookContext'`)
- **Namespace-module pattern**: `export namespace HookContext { export class Service ... ; export const layer ... }`

### Module Structure

Every source file follows:

1. Module-level JSDoc with `@since` tag
2. Imports (ordered as above)
3. Sections separated by `// ---...--- // Section Name // ---...---`
4. Exported functions with JSDoc (`@since`, `@example`)
5. Internal helpers marked `/** @internal */`

### Types

- `readonly` on all interface fields and function parameters
- `ReadonlyArray<T>` instead of `T[]`
- No `any`, no `@ts-ignore`, no non-null assertions (`!`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpsuesser/effect-claudecode](https://github.com/mpsuesser/effect-claudecode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
