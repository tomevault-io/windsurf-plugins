---
trigger: always_on
description: OpenCode plugin for autonomous penetration testing via Docker-based security tools.
---

# AGENTS.md — opencode-shannon-plugin

OpenCode plugin for autonomous penetration testing via Docker-based security tools.

## Build & Run Commands

```bash
bun run build          # Bundle with bun + emit declarations with tsc
bun run clean          # rm -rf dist
bun run typecheck      # tsc --noEmit (type-check only, no output)
bun run test           # bun test (no test files exist yet)
bun run clean && bun run build   # Full clean rebuild
```

Build is two-step: `bun build src/index.ts --outdir dist --target bun --format esm` then `tsc --emitDeclarationOnly`.

There are no tests, no linter, and no formatter configured. Rely on `tsc --noEmit` for correctness.

## Project Structure

```
src/
  index.ts              Plugin entry — registers all tools and hooks
  types.ts              Shared interfaces (ShannonToolInput, ShannonToolOutput, etc.)
  system-prompt.ts      System prompt constant injected into chat
  tools/                One directory per tool (14 tools total)
  hooks/                Hook implementations (authorization, progress, session)
  config/               Zod-based config schema + loader
  docker/               DockerManager singleton class
  commands/             CLI command definitions
  skills/               Markdown skill files (pentest guides)
```

### Tool directory layout

Full tools have 2–4 files:

```
src/tools/shannon-recon/
  index.ts              Barrel re-export (one line)
  tools.ts              Factory function(s) returning ToolDefinition
  types.ts              Tool-specific arg/result interfaces (optional)
  constants.ts          Description string constants (optional)
```

Simple tools (logic-audit, cloud-recon, api-fuzzer) put everything in `index.ts`.

## TypeScript Configuration

- **Target/Module**: ESNext, ESM (`"type": "module"` in package.json)
- **Module resolution**: `bundler`
- **Strict mode**: enabled (`strict: true`)
- **verbatimModuleSyntax**: enabled — use `import type` for type-only imports
- **noUncheckedIndexedAccess**: enabled — indexed access returns `T | undefined`
- **noImplicitOverride**: enabled
- **Types**: `bun-types` (not `@types/node`)

## Code Style

### Formatting

- **Indentation**: 2 spaces
- **Quotes**: double quotes (`"..."`)
- **Semicolons**: omitted (no trailing semicolons)
- **Trailing commas**: yes, in multi-line objects/arrays/params
- **Max line length**: not enforced, but keep readable (~100-120 chars)

### Imports

Order (separated by blank line when mixing groups):

1. External packages (`@opencode-ai/plugin`, `zod`, `picocolors`)
2. Internal absolute (`../../types`, `../../docker`)
3. Local relative (`./tools`, `./types`, `./constants`)

```typescript
import { tool, type ToolDefinition } from "@opencode-ai/plugin"
import type { ShannonToolInput } from "../../types"
import { SHANNON_RECON_DESCRIPTION } from "./constants"
```

Use `import type { ... }` for type-only imports — enforced by `verbatimModuleSyntax`.

Use `import * as fs from "fs"` for Node builtins (some files use `node:` prefix, but this is inconsistent — match the file you're editing).

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Tool factory | `createShannonXxx` | `createShannonRecon()` |
| Hook factory | `createShannonXxxHook` | `createShannonProgressTrackerHook()` |
| Tool name (registered) | `snake_case` | `shannon_recon`, `shannon_idor_test` |
| Interfaces/Types | `PascalCase` | `ShannonToolOutput`, `DockerExecutionResult` |
| Constants | `UPPER_SNAKE_CASE` | `SHANNON_RECON_DESCRIPTION` |
| Helper functions | `camelCase` (module-level, not exported) | `formatAutoResults()` |
| Directory names | `kebab-case` | `shannon-auth-session/` |
| Config schema | `PascalCase` + `Schema` suffix | `ShannonConfigSchema` |
| Zod import alias | always `z` | `import { z } from "zod"` |

### Tool Implementation Pattern

Every tool follows this exact factory pattern:

```typescript
import { tool, type ToolDefinition } from "@opencode-ai/plugin"

export function createShannonXxx(): ToolDefinition {
  return tool({
    description: "Short description of what the tool does.",
    args: {
      target: tool.schema.string().describe("The target URL or IP"),
      command: tool.schema.string().describe("The command to execute"),
      timeout: tool.schema.number().optional().describe("Timeout in ms"),
      mode: tool.schema.enum(["manual", "auto"]).optional().describe("Testing mode"),
    },
    async execute(args) {
      const output: string[] = []
      // ... build output
      return output.join("\n")
    },
  })
}
```

Key rules:
- Args defined with `tool.schema.*().describe("...")` — every arg needs `.describe()`
- Enum args: `tool.schema.enum(["value1", "value2"])`
- Object args: `tool.schema.object({ key: tool.schema.string() }).optional()`
- Return type is always `string` (output lines joined with `"\n"`)
- Output uses markdown formatting (`##`, `**`, `` ` ``, `---`)

### Barrel Exports

Every `index.ts` is a single-line re-export:

```typescript
export { createShannonRecon } from "./tools"
export type { ShannonReconArgs } from "./types"
```

### Error Handling

```typescript
// In tool execute functions — return error as string, never throw
try {
  // ... operation
} catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vichhka-git/opencode-shannon-plugin](https://github.com/vichhka-git/opencode-shannon-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
