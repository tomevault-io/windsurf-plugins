---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
nvm use                  # Node is pinned in .nvmrc (v22.21.1)
npm run build            # tsc -> build/, then chmod +x build/index.js
npm run watch            # tsc --watch
npm test                 # vitest in watch mode
npm run test:precommit   # vitest run (single pass) — what CI and the pre-commit hook use
npm run lint             # eslint src scripts
npm run lint:fix
npm run format           # prettier --write src/**/*.ts
npm run inspector        # MCP Inspector against build/index.js (build first)
```

Run a single test file or case:

```bash
npx vitest run src/tools/get-book-by-id/index.test.ts
npx vitest run -t "should return book details when given a valid OLID"
```

`vitest.config.ts` sets two things:

- `exclude` adds `build/**` to the defaults. Without it, `tsc` compiles the `.test.ts` files into `build/`, Vitest collects those copies alongside the sources, and every test runs twice locally — against stale compiled output, since `tsc` never removes artifacts for deleted source files. Collection is otherwise the Vitest default: `src/**/*.test.ts` plus the release scripts' `scripts/*.test.mjs`.
- `silent: "passed-only"` hides console output from passing tests. The handlers `console.error` in their catch blocks, so the error-path tests otherwise bury the results under stack traces. **Consequence:** a `console.log` added to debug a *passing* test prints nothing. Make the test fail, or run that file with `--silent=false`.

The husky pre-commit hook runs `lint-staged` (eslint --fix + prettier) followed by the **full** test suite, so commits are slow but pre-verified.

## Architecture

An MCP stdio server wrapping the Open Library HTTP API. Three layers:

**`src/index.ts`** — the `OpenLibraryServer` class. Builds the two `axios` clients via `createOpenLibraryClients`, then drives both request handlers from `TOOLS`: `ListTools` maps the registry through `toInputSchema`, and `CallTool` looks the tool up by name. Reads its version at runtime from `../package.json` relative to `import.meta.url` (resolves to the package root from `build/index.js`). The `run()` call is gated on `process.argv[1] === new URL(import.meta.url).pathname` so importing the module in tests doesn't start a transport. `run()` also owns the `SIGINT` handler: the constructor deliberately installs no process-wide listener, because the suite builds a server per test and those listeners accumulate (`src/index.test.ts` asserts construction adds none).

**`src/tools/<tool-name>/`** — one directory per tool, each with `index.ts` (handler, zod arg schema, and the exported `ToolDefinition`), optional `types.ts` (Open Library API response shapes), and `index.test.ts`. `src/tools/registry.ts` collects the definitions into `TOOLS`; `src/tools/index.ts` re-exports everything.

**`src/utils/`** — `http.ts` (client factory; User-Agent and 15s timeout), `errors.ts` (`parseArgs`, `isNotFound`, `describeError`, `toErrorResult`), `results.ts` (`textResult`, `errorTextResult`, `jsonResult`), `schema.ts` (`toInputSchema`), `search.ts` (shared `/search.json` projection and paging schemas), `covers.ts` (cover existence check). `src/test-support/` holds test-only helpers — currently `axiosErrorWithStatus`, the one place the shape of an axios failure is constructed.

Every handler has the same signature: `(args: unknown, clients: OpenLibraryClients)`, where `clients` is `{ api, covers }` — `api` is based at `https://openlibrary.org`, `covers` at `https://covers.openlibrary.org`. The uniform signature is what lets `CallTool` dispatch generically.

### Tool schemas are generated from zod

A tool's input contract is declared **once**, as a zod schema. `toInputSchema` (`src/utils/schema.ts`) converts it with `z.toJSONSchema(schema, { io: "input" })` for the `ListTools` response. Two things to know:

- **`io: "input"` is mandatory.** The default (`"output"`) throws on any schema containing a transform, and marks defaulted fields as `required`.
- **`.refine()` is silently dropped.** Cross-field rules (like `search_books` requiring at least one criterion) must be repeated in the tool's `description`, or clients never learn about them.

Because `io: "input"` reports the *pre*-transform type, a `z.string().transform(...).pipe(z.enum([...]))` would publish a bare `{type: "string"}` and lose the enum. `get_book_by_id` therefore declares a plain enum and lowercases `idType` before calling `parseArgs`.

Adding a tool means two edits: a new directory under `src/tools/`, and an entry in `TOOLS` in `src/tools/registry.ts`. `src/index.test.ts` derives its assertions from `TOOLS`, so the only test change is refreshing the schema snapshot with `npx vitest run -u`.

### Error convention

The rule comes from the spec's `CallToolResult.isError` docs: errors originating from a tool "SHOULD be reported inside the result object, with `isError` set to `true`, *not* as an MCP protocol-level error response. Otherwise, the LLM would not be able to see that an error occurred and self-correct." Only failures in *finding* a tool stay protocol-level.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [8enSmith/mcp-open-library](https://github.com/8enSmith/mcp-open-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
