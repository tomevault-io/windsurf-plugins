---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

An **MCP (Model Context Protocol) server** that lets AI agents drive the [JHipster](https://github.com/jhipster/generator-jhipster) CLI to scaffold and evolve applications via **JDL** (JHipster Domain Language). The server spawns the user's **global `jhipster` binary**; it does not bundle the generator.

Transport is **stdio**. The entry point is [src/index.ts](src/index.ts).

## Documentation

User-facing documentation lives in [docs/](docs/) (a numbered, progressive guide for JHipster users new to MCP — installation, tools/resources/prompts reference, context management, advanced usage, and an engine page that mirrors this file). **Keep it in sync as features land:** when a roadmap item ships, update the relevant `docs/*.md` page alongside the code, the same way you update the README. The roadmap itself is [docs/ROADMAP.md](docs/ROADMAP.md).

## Commands

```bash
npm run build       # tsup → dist/index.js (the published binary)
npm run dev         # tsup --watch
npm run typecheck   # tsc --noEmit (checks src AND test)
npm test            # node --test via tsx, runs test/**/*.test.ts
npm run test:watch  # tests in watch mode
npm start           # node dist/index.js (must build first)
```

Always run `npm run typecheck` and `npm test` before considering a change done.

## Architecture

```
src/
  index.ts            # registers all tools + resources + prompts, connects stdio transport
  jhipster.ts         # spawn wrapper around the `jhipster` CLI + result formatting
  apply.ts            # applyJdl() — shared persist-vs-dry-run + result formatting
  progress.ts         # makeProgressReporter() — streams output as progress notes
  result.ts           # toStructuredResult() + shared outputSchema (parses generator output)
  jdl/builders.ts     # pure JDL string builders + quickLintJdl, strict name validation
  tools/*.ts          # one MCP tool per file, each exports register<Name>(server)
  resources/*.ts      # one MCP resource per file, each exports register<Name>(server)
  prompts/*.ts        # one MCP prompt per file, each exports register<Name>(server)
```

Data flow: a JDL-applying tool builds its JDL, then calls `applyJdl()` ([src/apply.ts](src/apply.ts)), which either persists the `.jdl` into `workingDirectory` and runs `jhipster jdl <file> --force --skip-git`, or (when `dryRun`) calls `runJdlIsolated()` — generate in a throwaway temp dir and discard. All five applying tools (`create_app_from_jdl`, `import_jdl`, `add_entity`, `add_relationship`, `set_option`) share this and expose a `dryRun` flag. `validate_jdl` always isolates: local `quickLintJdl` first, then `runJdlIsolated`.

**Important — dry-run is isolation, not a flag.** JHipster 9's `--dry-run` only *prints conflicts*; it still writes files (verified against real CLI). So a true no-write preview means generating in a temp dir. `runJdlIsolated` copies the project's `.yo-rc.json` + `.jhipster/` (entity configs) into the temp dir for context, runs `jhipster jdl preview.jdl --force --skip-git --skip-install` there, then `rm`s it. Never pass `--dry-run` expecting no writes.

## Conventions (match these)

- **ESM with `.js` import extensions** in TypeScript source (e.g. `import { runJhipster } from "../jhipster.js"`). Required for Node ESM resolution; tsup/tsx honor it.
- **Tool schemas use zod raw shapes** (a plain object of zod validators), passed as `inputSchema` to `server.registerTool(name, { title, description, inputSchema }, handler)`.
- **Every tool takes an absolute `workingDirectory`.** The server has no notion of a cwd. Validate it via `assertDirectoryExists` (already done inside `runJhipster`).
- **Non-interactive always.** Append `--force --skip-git` to every generator invocation; the spawn wrapper sets `CI=true`. Never run `jhipster` interactively.
- **Spawn with `shell: false`** — no shell. Validate any user-influenced args (see `run_jhipster`'s allowlist + `ARG_PATTERN`).
- **JDL injection guard:** entity/field/type/package names are validated against strict regex in [src/jdl/builders.ts](src/jdl/builders.ts). Reuse those builders rather than hand-concatenating JDL.
- Handlers return `{ isError, content: [{ type: "text", text }] }`. A thrown error is also surfaced by the SDK as an `isError` result.
- **Structured output:** every jhipster-running tool declares `outputSchema: structuredOutputShape` and returns `structuredContent: toStructuredResult(result, { jdl?, dryRun? })`. The SDK validates `structuredContent` against the schema **only on non-error results** — pre-spawn guard failures (allowlist, empty-dir, lint) stay `isError` with no `structuredContent` and are fine. `StructuredRunResult` has an index signature so it satisfies the SDK's `{ [x: string]: unknown }` requirement.

## Adding a new tool

1. Create `src/tools/<name>.ts` exporting `register<Name>(server: McpServer)`.
2. If it emits JDL, add/reuse a builder in `src/jdl/builders.ts` (with name validation).
3. Register it in [src/index.ts](src/index.ts).
4. Add a test in `test/tools/<name>.test.ts` (pattern below).

## Testing approach

- Runner: Node's built-in `node:test` driven by `tsx` — **no Jest/Vitest.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jhipster/jhipster-mcp](https://github.com/jhipster/jhipster-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
