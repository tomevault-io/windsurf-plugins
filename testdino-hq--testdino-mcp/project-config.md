---
trigger: always_on
description: You are working on an MCP (Model Context Protocol) server that connects TestDino to AI agents (Cursor, Claude Code, VS Code, etc.). This is an npm package published as `testdino-mcp`. Users install it and configure it in their AI tool to interact with TestDino's test data via natural language.
---

# CLAUDE.md — TestDino MCP Server

You are working on an MCP (Model Context Protocol) server that connects TestDino to AI agents (Cursor, Claude Code, VS Code, etc.). This is an npm package published as `testdino-mcp`. Users install it and configure it in their AI tool to interact with TestDino's test data via natural language.

Read this file fully before doing anything. Follow it without deviation.

---

## Project Identity

- **What**: MCP server (stdio transport, not HTTP)
- **Language**: TypeScript (strict mode, ES modules)
- **Runtime**: Node.js >= 20
- **Published to**: npm (`testdino-mcp`)
- **API**: Talks to `https://api.testdino.com` using Bearer token auth (PAT)
- **Users**: Developers and QA engineers using AI coding tools

---

## Before You Write Any Code

1. Read this file completely
2. Understand the tool you're modifying — read its source file and the corresponding section in `docs/TOOLS.md`
3. Check existing patterns in `src/tools/` — follow them exactly, don't invent new conventions
4. If adding a new tool, use `src/tools/testruns/list-testruns.ts` as the reference implementation

---

## Architecture

```
src/
├── index.ts              ← Server entry point (registers tools + resources)
├── lib/
│   ├── env.ts            ← API URL + PAT resolution
│   ├── endpoints.ts      ← All API endpoint URL builders (centralized)
│   ├── request.ts        ← HTTP fetch wrapper (apiRequest / apiRequestJson)
│   └── file-utils.ts     ← Local file reading, base64 encoding, step validation
└── tools/
    ├── health.ts         ← PAT validation + account info
    ├── index.ts          ← Barrel export for all tools
    ├── testruns/         ← list-testruns, get-run-details
    ├── testcases/        ← list-testcase, get-testcase-details, debug-testcase
    ├── manual-testcases/ ← CRUD for manual test cases
    └── manual-testsuites/← list + create test suites
```

**Data flow for every tool call:**

```
AI agent calls tool → index.ts routes by name → handler validates args
→ getApiKey() resolves PAT → endpoints.X() builds URL → apiRequestJson() fetches
→ handler formats response → returns { content: [{ type: "text", text: ... }] }
```

---

## Tool Development Conventions

Every tool file exports exactly two things:

1. **Tool definition** — `export const xyzTool = { name, description, inputSchema }`
2. **Handler function** — `export async function handleXyz(args) { ... }`

### Tool Definition Rules

- `name`: snake_case (e.g., `list_testruns`, `get_run_details`)
- `description`: Write for AI agents, not humans. Be specific about what the tool returns and when to use it. Include usage hints
- `inputSchema`: JSON Schema format. Every property MUST have a `description`. Use `enum` for known values. Mark `required` fields accurately
- Descriptions are the AI's only guide — vague descriptions = wrong tool usage

### Handler Rules

1. **Auth first**: Always call `getApiKey(args)` and throw if missing
2. **Validate required params**: Check presence, throw with specific message
3. **Build URL via endpoints.ts**: Never construct URLs manually in handlers
4. **Type conversions**: Use `String()` for string params, `Number()` for numeric. Be consistent
5. **Response format**: Return `{ content: [{ type: "text", text: JSON.stringify(response, null, 2) }] }`
6. **Error handling**: Wrap in try/catch, prefix errors with context: `throw new Error("Failed to [action]: [detail]")`

### Adding a New Tool — Checklist

1. Add endpoint URL builder in `src/lib/endpoints.ts`
2. Create tool file in the appropriate `src/tools/<category>/` directory
3. Export from `src/tools/index.ts`
4. Register in `src/index.ts` (add to tools array + add routing if-block)
5. Update `docs/TOOLS.md` with full documentation
6. Update `docs/skill.md` if it affects AI agent workflows
7. Run full verify: `npm run typecheck && npm run lint && npm run test`

---

## Code Rules

### TypeScript

- Strict mode enabled — no `any` unless absolutely necessary (use `Record<string, unknown>`)
- All imports use `.js` extensions (ES module requirement)
- Use named exports only — no default exports
- Keep handler functions focused: auth → validate → fetch → format → return

### Naming

- Tool files: `kebab-case.ts` (e.g., `list-testruns.ts`)
- Tool names (MCP): `snake_case` (e.g., `list_testruns`)
- Tool definition exports: `camelCaseTool` (e.g., `listTestRunsTool`)
- Handler exports: `handleCamelCase` (e.g., `handleListTestRuns`)
- Interfaces/types: `PascalCase` (e.g., `ListTestRunsArgs`)

### Error Messages

- Missing PAT: `"Missing TESTDINO_PAT environment variable. Please configure it in your .cursor/mcp.json file under the 'env' section."`
- Missing required param: `"[paramName] is required"`
- API failure: `"Failed to [action]: [error detail]"`
- Keep messages actionable — tell the user what to do, not just what went wrong

### What NOT to Do

- Don't construct API URLs directly in tool handlers — always use `endpoints.ts`
- Don't add new dependencies without strong justification

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [testdino-hq/testdino-mcp](https://github.com/testdino-hq/testdino-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
