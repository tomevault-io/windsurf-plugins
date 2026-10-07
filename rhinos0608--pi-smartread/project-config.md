---
trigger: always_on
description: These instructions apply to the repository root. Keep agent-specific operational rules here; put user-facing behavior and setup in `README.md`.
---

# Pi-SmartRead agent instructions

## Scope

These instructions apply to the repository root. Keep agent-specific operational rules here; put user-facing behavior and setup in `README.md`.

Pi-SmartRead is a TypeScript/ESM Pi extension plus standalone MCP server. It owns code retrieval, structural analysis, repository intelligence, language-server access (read-only except applyProposal), workspace evidence production, and the SmartRead side of the SmartEdit evidence/LSP proposal contract.

## Canonical commands

Use **npm**. `package-lock.json` is the canonical lockfile and CI runs Node 20.

| Task | Command |
|---|---|
| Install | `npm ci` |
| Typecheck | `npm run typecheck` |
| Lint | `npm run lint` |
| Full tests | `npm test` |
| Focused test | `npx vitest run path/to/test.ts` |
| LSP unit suite | `npx vitest run test/unit/lsp` |
| Validate skills | `node scripts/validate-skills.mjs` |
| Run MCP server | `npm run mcp-server` |
| Load extension locally | `pi -e ./src/index.ts` |
| Opt-in real LSP suite | `PI_SMARTREAD_LSP_CONFORMANCE=1 npx vitest run test/integration/lsp` |

Before finishing source changes, run the narrowest relevant tests plus `npm run typecheck`. For broad/runtime changes, run `npm test`. CI runs typecheck + full tests on Linux, macOS, and Windows.

## Current model-facing surfaces

### Pi extension

- `read`: already-known source content only; exactly one selector per call: `path`, `paths`, or `symbol`. Natural-language query mode does not exist.
- `inspect`: explicit `mode: "file" | "directory" | "script"`; structural/architectural analysis only, never LSP navigation.
- `find`: replaces Pi's builtin `find`; exact schema `{pattern, path?, limit?}`. Discovers files/directories by glob, fuzzy name, or natural-language description; discovery evidence only.
- `grep`: broad text/symbol/concept discovery, batch queries, structural options, graph filters. Optional judging applies only to natural-language smart-cascade queries and falls back to unjudged results on failure.
- `/judge`: user-level `off|local|cloud|status|install`; off by default. Cloud auth uses Pi's OpenRouter auth store; von install requires explicit UI confirmation.
- `LSP`: strict compiler/language-server semantics (read-only except applyProposal). Tool name is uppercase `LSP`.
- `skill`: discovers project/package/global skills.
- Experimental: `graph_mutate`, `git_notes_read`, `git_notes_write` only when enabled.

### Standalone MCP

The MCP registry exposes `inspect`, `find`, `grep`, `LSP`, `skill`, plus enabled experimental registry tools. It does **not** expose the Pi-only wrapped `read` tool. Natural-language judge mode is controlled by user environment settings.

MCP also exposes prompts from `src/mcp/mcp-prompts.ts` and `smartread://` resources from `src/mcp/mcp-resources.ts`.

## Coordinate conventions: do not mix them

There are two LSP-facing contracts:

- Strict `LSP` tool (Pi + standalone MCP): `position: { line, character }` is **0-based** in `server.positionEncoding`. Operations use names such as `goToDefinition`, `findReferences`, `codeActions`.
- Script-mode `lsp.*` helpers are an internal composition API: their navigation `line` / `character` inputs are **1-based** and use helper operation names such as `definition`, `references`, `implementation`.

Do not port examples between the strict tool and script host helpers without converting both operation names and coordinates. Public `inspect` has no navigation/diagnostics surface.

## Source routing

| Area | Start here |
|---|---|
| Extension bootstrap/lifecycle | `src/index.ts`, `src/extension-lifecycle.ts`, `src/extension-registration.ts` |
| Wrapped reads/evidence/enrichment | `src/hook.ts`, `src/read/`, `src/evidence/` |
| Inspect modes | `src/inspect/inspect-tool.ts`, `src/inspect/inspect.ts`, `src/script-mode/` |
| Grep/search | `src/search/grep-tool.ts`, `src/search/grep-cascade.ts`, `src/search/grep-structural-executor.ts`, `src/search/find-tool.ts` |
| Optional relevance judge | `src/judge/`, `src/search/grep-judge-stage.ts` |
| Strict LSP | `src/lsp/lsp-tool.ts`, `src/lsp/lsp-strict-contract.ts`, `src/lsp/lsp-operation-registry.ts`, `src/lsp/lsp-executor.ts` |
| LSP runtime/install/trust | `src/language-intelligence/`, `src/lsp/lsp-manager.ts`, `src/lsp/lsp-connection.ts` |
| Repository intelligence | `src/repository/`, `src/graph/`, `src/indexing/`, `src/retrieval/` |
| MCP | `src/mcp-server.ts`, `src/mcp-registry.ts`, `src/mcp/` |
| Skills | `skills/`, `src/runtime/skill-tool.ts`, `scripts/validate-skills.mjs` |

## Evidence contract and SmartEdit boundary

`@rhinos0608/pi-workspace-protocol` is pinned in `package.json`. Import `PROTOCOL_SCHEMA_VERSION`; do not hardcode a schema number in new code.

- `read` produces strong evidence only for content actually rendered. Complete file blocks can authorize patching; omitted/partial packed blocks cannot.
- Large-file AST outlines produce line-range evidence, not full-file authority.
- `inspect`, `find`, and `grep` produce discovery/search-match evidence. Read the relevant file/range before mutation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-SmartRead](https://github.com/rhinos0608/Pi-SmartRead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
