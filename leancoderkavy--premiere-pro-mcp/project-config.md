---
trigger: always_on
description: Canonical instructions for any IDE or coding agent working in this repository.
---

# AGENTS.md

Canonical instructions for any IDE or coding agent working in this repository.
Cursor, GitHub Copilot, Codex, Claude Code, Gemini CLI, Windsurf, and similar tools should start here.

This is **not** the product website LLM crawl file. That file is <https://premiere-pro-mcp.com/llms.txt>, generated in the separate website repository.

Adapter stubs (`CLAUDE.md`, `GEMINI.md`, `CONVENTIONS.md`, `.github/copilot-instructions.md`, `.cursor/rules/`, `.windsurf/rules/`, `.clinerules`, `.continue/rules/`, `.junie/guidelines.md`) point here. Keep those stubs thin; put durable guidance in this file or in a focused Cursor/Copilot path rule. Optional Cursor rule `.cursor/rules/jev-surfaces.mdc` scores CEP / UXP / claims file candidates. It is not a TypeSafe Jev sidecar protocol; do not add one here.

## What this repository is

Independent TypeScript [MCP](https://modelcontextprotocol.io) server that gives a compatible AI client structured control over supported **Adobe Premiere Pro** workflows. Display name: MCP for Adobe Premiere Pro. npm package: `premiere-pro-mcp`. MCP server name: `io.github.leancoderkavy/premiere-pro`.

It is not affiliated with Adobe. The only npm package published from this repo is `premiere-pro-mcp`.

Production path: local Node.js MCP server (stdio) talking to a **CEP** panel inside Premiere through private file-based IPC (`.jsx` command files and `.json` responses). **UXP** is a capability-aware preview backend for Premiere 25.6+ documented APIs. A failed UXP mutation must never be silently retried through CEP or the undocumented QE DOM.

The local stdio server is the product. The HTTP `/mcp` endpoint is an operator-managed transport, not a public remote-Premiere product. See `docs/hosted-mcp-product-boundary.md`.

## Canonical sources

Read only what the task needs. Current source and release metadata beat dated snapshots.

| Topic | Source |
| --- | --- |
| Product, setup, architecture overview | `README.md` |
| This agent map | `AGENTS.md` (this file) |
| Trust model and vulnerability reports | `SECURITY.md` |
| Human contributor workflow | `CONTRIBUTING.md` |
| API research (dated; not live counts) | `RESEARCH.md` |
| Registered tool catalog | `docs/supported-actions.md` (generated) |
| Product claims governance | `docs/claims-registry.md`, `docs/claims-registry.json` |
| Published-package facts | npm tarball, synced by [`leancoderkavy/premiere-pro-mcp-site`](https://github.com/leancoderkavy/premiere-pro-mcp-site) to <https://premiere-pro-mcp.com/facts/> |
| Development source metadata | `release-metadata.json` |
| Release titles, tags, and versions | `docs/release-conventions.md` |
| MCP runtime instructions for clients | `src/workflows/agent-instructions.ts` |
| Hosted HTTP boundary | `docs/hosted-mcp-product-boundary.md` |
| UXP 26.3 coverage | `docs/adobe-uxp-26.3-coverage.md` |
| Claude Code develop skill | `claude-plugins/premiere-pro/skills/develop-premiere-pro-mcp/SKILL.md` |

Do not mix published-package counts with development-source counts. `docs/supported-actions.md` is the source catalog; the published npm artifact is what the website reports. In this repo the README `### Latest release:` heading anchors client version pins. A listed tool is not proof that a particular Premiere host supports it.

## Architecture

```text
AI client  --stdio MCP-->  Node server (src/server.ts)
                              |  CEP: write cmd_*.jsx, poll JSON  -->  cep-plugin  -->  ExtendScript / QE DOM
                              |  UXP: authenticated WebSocket     -->  uxp-plugin  -->  documented Premiere UXP
                              |  AE:  optional CEP                -->  after-effects-cep-plugin (MOGRT studio)
```

- `src/index.ts` — stdio entry.
- `src/http-server.ts` — Streamable HTTP `/mcp`. Requires `MCP_AUTH_TOKEN` in production.
- `src/server.ts` — registers tools, resources, prompts; filters by authority and tool packs.
- `src/tools/` — one module per capability area; each exports `getXTools(...)`.
- `src/bridge/script-builder.ts` — ES3 ExtendScript helpers and `escapeForExtendScript`.
- `src/bridge/file-bridge.ts` — private temp-dir IPC for CEP.
- `src/bridge/uxp-websocket-bridge.ts` — authenticated UXP transport.
- `src/security/capabilities.ts` — `inspect`, `edit`, `export`, `filesystem`, `unsafe-script`.
- `src/ai/` — local planning/preview helpers. They do not call Adobe generative APIs.
- `src/workflows/` — tool packs, structured results, client-facing instructions, prompt catalog.
- `cep-plugin/` — production Premiere CEP panel.
- `uxp-plugin/` — Premiere 25.6+ UXP panel (CommonJS, Adobe ESLint plugin).
- `after-effects-cep-plugin/` — separate AE connector for guarded MOGRT recipes.
- `tests/` — Vitest. Mocks the bridge; does not prove live Premiere behavior.
- `scripts/` — inventories, catalogs, installers, publish helpers.

Premiere metadata is several host surfaces. Documented UXP `Metadata.*` methods
are already mapped. Project-panel column JSON, Premiere-private project metadata
XML, and file/clip XMP are distinct; `premiere://project/metadata` is only a
path-redacted project/timeline summary. Field-level inspect/update uses
`uxp.XMPMeta` / AdobeXMPScript with a 256-field cap, sensitive-EXIF omit, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leancoderkavy/premiere-pro-mcp](https://github.com/leancoderkavy/premiere-pro-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
