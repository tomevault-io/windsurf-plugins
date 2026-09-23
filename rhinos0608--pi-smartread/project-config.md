---
trigger: always_on
description: These instructions apply to the repository root. Keep agent-specific operational rules here; put user-facing behavior and setup in `README.md`.
---

# Pi-SmartRead agent instructions

## Scope

These instructions apply to the repository root. Keep agent-specific operational rules here; put user-facing behavior and setup in `README.md`.

Pi-SmartRead is a TypeScript/ESM Pi extension plus standalone MCP server. It owns code retrieval, structural analysis, repository intelligence, read-only language-server access, workspace evidence production, and the SmartRead side of the SmartEdit evidence/LSP proposal contract.

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

- `read`: exactly one selector per call: `path`, `paths`, `query`, or `symbol`.
- `inspect`: explicit `mode: "file" | "directory" | "navigate" | "script"`; mode is never inferred.
- `grep`: text/symbol/concept search, batch queries, structural options, graph filters.
- `LSP`: strict read-only LSP contract. Tool name is uppercase `LSP`.
- `skill`: discovers project/package/global skills.
- Experimental: `graph_mutate`, `git_notes_read`, `git_notes_write` only when enabled.

### Standalone MCP

The MCP registry exposes `inspect`, `grep`, `skill`, plus enabled experimental registry tools. It does **not** expose the Pi-only wrapped `read` tool or the Pi-registered strict `LSP` tool.

MCP also exposes prompts from `src/mcp/mcp-prompts.ts` and `smartread://` resources from `src/mcp/mcp-resources.ts`.

## Coordinate conventions: do not mix them

There are two LSP-facing contracts:

- Strict `LSP` tool: `position: { line, character }` is **0-based** in `server.positionEncoding`. Operations use names such as `goToDefinition`, `findReferences`, `codeActions`.
- `inspect { mode: "navigate" }` and script-mode `lsp.*` helpers: navigation `line` / `character` are **1-based** and use the inspect navigation operation names such as `definition`, `references`, `implementation`.

Do not port examples between these surfaces without converting both operation names and coordinates.

## Source routing

| Area | Start here |
|---|---|
| Extension bootstrap/lifecycle | `src/index.ts`, `src/extension-lifecycle.ts`, `src/extension-registration.ts` |
| Wrapped reads/evidence/enrichment | `src/hook.ts`, `src/read/`, `src/evidence/` |
| Inspect modes | `src/inspect/inspect-tool.ts`, `src/inspect/inspect.ts`, `src/script-mode/` |
| Grep/search | `src/search/grep-tool.ts`, `src/search/grep-cascade.ts`, `src/search/grep-structural-executor.ts` |
| Strict LSP | `src/lsp/lsp-tool.ts`, `src/lsp/lsp-strict-contract.ts`, `src/lsp/lsp-operation-registry.ts`, `src/lsp/lsp-executor.ts` |
| LSP runtime/install/trust | `src/language-intelligence/`, `src/lsp/lsp-manager.ts`, `src/lsp/lsp-connection.ts` |
| Repository intelligence | `src/repository/`, `src/graph/`, `src/indexing/`, `src/retrieval/` |
| MCP | `src/mcp-server.ts`, `src/mcp-registry.ts`, `src/mcp/` |
| Skills | `skills/`, `src/runtime/skill-tool.ts`, `scripts/validate-skills.mjs` |

## Evidence contract and SmartEdit boundary

`@rhinos0608/pi-workspace-protocol` is pinned in `package.json`. Import `PROTOCOL_SCHEMA_VERSION`; do not hardcode a schema number in new code.

- `read` produces strong evidence only for content actually rendered. Complete file blocks can authorize patching; omitted/partial packed blocks cannot.
- Large-file AST outlines produce line-range evidence, not full-file authority.
- `inspect` and `grep` produce discovery/search-match evidence. Read the relevant file/range before mutation.
- `details.workspaceEvidence` on tool results is the durable source of truth. The in-memory resolver cache is derived from tool-result events.
- Evidence-producing canonical paths must use real paths with symlinks resolved. Follow the existing `tryCanonical`/realpath pattern; do not replace it with bare `path.resolve`.
- SmartRead publishes evidence and semantic proposals. **SmartEdit owns file mutation.** SmartRead must never apply LSP WorkspaceEdits.

See `docs/lsp-smartedit-contract.md` before changing language-intelligence RPC proposal behavior.

## LSP invariants

- `LSP` is strict and fail-closed: unknown operations/foreign fields are caller errors.
- Exact `server` routing means exact descriptor id. No fuzzy server selection.
- Unroutable requests return an `unavailable` envelope; do not invent a fallback server.
- Raw `request` is observational allowlist only. `workspace/executeCommand`, `workspace/applyEdit`, and unknown methods are rejected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhinos0608/Pi-SmartRead](https://github.com/rhinos0608/Pi-SmartRead) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
