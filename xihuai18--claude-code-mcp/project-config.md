---
trigger: always_on
description: This repository is a TypeScript (ESM) MCP server wrapping Claude Agent SDK / Claude Code CLI.
---

# Repo Agent Instructions (claude-code-mcp)

This repository is a TypeScript (ESM) MCP server wrapping Claude Agent SDK / Claude Code CLI.
Package: `@leo000001/claude-code-mcp`.
Assumption: MCP server and client run on the same machine (same platform), via stdio.

> Last Updated: 2026-02-27

## Document Boundary (Must Read)

This repo intentionally separates **execution rules** from **design details**:

| Topic                                                            | Primary Doc      |
| ---------------------------------------------------------------- | ---------------- |
| How to execute work, upgrade flow, required checks               | `AGENTS.md`      |
| Architecture internals, field/message mapping, lifecycle details | `docs/DESIGN.md` |
| End-user usage guide                                             | `README.md`      |
| Release history                                                  | `CHANGELOG.md`   |

Non-negotiable dedupe rule:

- Keep `AGENTS.md` action-oriented.
- Keep long parameter tables, message mapping, and protocol deep dives in `docs/DESIGN.md`.
- If details are needed, link to DESIGN anchors instead of duplicating.

Visibility rule for MCP-facing guidance:

- Do not assume MCP-connected coding agents can read `README.md`, `AGENTS.md`, or `docs/DESIGN.md`.
- Put protocol-critical runtime guidance first in MCP tool descriptions and MCP resources.
- Treat repo docs as human/maintainer guidance unless a client explicitly injects them into model context.

## Design Snapshot (Summary Only)

Project direction:

- Use local Claude settings by default (`settingSources: ["user","project","local"]`)
- Expose only 4 MCP tools (`claude_code`, `claude_code_reply`, `claude_code_session`, `claude_code_check`)
- Keep startup non-blocking (start/reply return quickly; poll with check)
- Provide three-layer permission control (`advanced.tools` + allow/deny + async decision)

Detailed behavior, full field semantics, and lifecycle mapping:

- `docs/DESIGN.md#sdk-interface-baseline`
- `docs/DESIGN.md#upgrade-methodology`
- `docs/DESIGN.md` Section 4 (Options mapping matrix)
- `docs/DESIGN.md` Section 5 (SDK message mapping matrix)

## SDK Upgrade Runbook (Execution Playbook)

When dependency interfaces change (`@anthropic-ai/claude-agent-sdk`, `@modelcontextprotocol/sdk`, sometimes zod schema impacts), execute in this order:

1. Confirm authoritative type definitions:
   - `node_modules/@anthropic-ai/claude-agent-sdk/sdk.d.ts`
   - relevant MCP SDK type surface in `node_modules/@modelcontextprotocol/sdk`
2. Compare against design matrices in `docs/DESIGN.md` (Sections 4 and 5).
3. Apply code updates to required touch points:
   - `src/server.ts` (zod schema and tool contract)
   - `src/utils/build-options.ts` (Option field mapping/defaults)
   - `src/tools/query-consumer.ts` (message mapping/permission callback behavior)
   - `src/session/manager.ts` (session and permission lifecycle)
   - `src/types.ts` (shared types/const tuples)
4. Sync docs and release notes:
   - `README.md`
   - `docs/DESIGN.md`
   - `AGENTS.md`
   - `CHANGELOG.md`
5. Run full checks:
   - `npm run typecheck`
   - `npm run lint`
   - `npm test`
   - `npm run format:check`

## Full Maintenance & Dependency Update Workflow

This section is the authoritative end-to-end workflow for updating dependencies and keeping code + docs + tests aligned.

### 0) Goals and Constraints

- **Goal:** keep MCP tool contracts stable while staying current with the upstream SDKs.
- **Rule:** SDK type definitions are authoritative (`sdk.d.ts`), changelogs are hints.
- **Rule:** avoid duplicating long tables here; update detailed matrices only in `docs/DESIGN.md`.
- **Rule:** do not ship stale lockfiles when `package.json` uses `^` ranges (users will resolve newer versions).
- **Hygiene:** any temporary audit artifacts must be deleted before finishing.

### 1) Detect Updates (Local, Reproducible)

Run:

- `npm outdated` (top-level)
- `npm outdated --all` (transitive signal only; don't chase majors unless needed)

Record:

- current / wanted / latest for `@anthropic-ai/claude-agent-sdk` and `@modelcontextprotocol/sdk`
- whether `zod` stays compatible (SDK peers `zod@^4`)

### 2) Establish the Interface Baseline (Authoritative)

Always treat the installed type surface as the source of truth:

- Claude Agent SDK: `node_modules/@anthropic-ai/claude-agent-sdk/sdk.d.ts`
- MCP SDK: `node_modules/@modelcontextprotocol/sdk/dist/**` (only the surfaces we import)

If you need to compare two versions without changing the workspace yet:

1. Create a temporary directory under `tmp/` (example: `tmp/deps-audit/`).
2. Download tarballs via `npm pack` (example):
   - `npm pack @anthropic-ai/claude-agent-sdk@<old>`
   - `npm pack @anthropic-ai/claude-agent-sdk@<new>`
3. Extract and diff `package/sdk.d.ts`.
4. Delete `tmp/deps-audit/` after the report is done.

### 3) Impact Analysis Checklist (What Can Break)

For each upgraded runtime dependency:

- **Options surface:** compare SDK `Options` fields to `src/utils/build-options.ts` (`OptionSource` + copy logic).
- **Message surface:** compare SDK `SDKMessage` union (new `type`/`subtype`) to `src/tools/query-consumer.ts` mapping.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xihuai18/claude-code-mcp](https://github.com/xihuai18/claude-code-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
