---
trigger: always_on
description: Zowe MCP is a Model Context Protocol (MCP) server and VS Code extension that provides AI tools for interacting with z/OS systems. It enables LLMs to work with data sets, jobs, and UNIX System Services on one or more z/OS systems.
---

# Zowe MCP - Agent Instructions

## Project Overview

Zowe MCP is a Model Context Protocol (MCP) server and VS Code extension that provides AI tools for interacting with z/OS systems. It enables LLMs to work with data sets, jobs, and UNIX System Services on one or more z/OS systems.

## Repository Structure

This is an npm workspaces monorepo with four packages:

- `packages/zowe-mcp-common` — Shared utilities (CJS, consumed by all other packages). Exports `plural()`.
- `packages/zowe-mcp-server` — Standalone MCP server (ESM, npm **`@zowe/mcp-server`**)
- `packages/zowe-mcp-vscode` — VS Code extension that registers the server (CommonJS) and contributes Zowe and ISPF color themes (`themes/*.json`)
- `packages/zowe-mcp-evals` — AI evaluations: runs an LLM agent against the MCP server (mock or native), checks tool choice/arguments and answer content, produces a Markdown report. Uses Vercel AI SDK and MCP SDK client. Config: gitignored `evals.config.json` at **repo root** (vLLM, Gemini, or LM Studio). Supports single-model (legacy) or **multi-model** (`models` array with `id` per entry; first is default). Use `--model <id>` to select a model when using multi-model config. LM Studio provider (`"lmstudio"`) uses the OpenAI-compatible endpoint (default `http://localhost:1234/v1`); the runner auto-loads the model via `POST /api/v1/models/load` with `contextLength` (default 32768) so the context window is large enough for tool definitions; when `serverModel` is missing, `loadEvalsConfig` queries `GET /v1/models` and lists available models in the error message. Run from repo root: `npm run evals` (options after `--`). Question sets: YAML files in `questions/` with per-set `repetitions`, `minSuccessRate`, optional `mock`/`native`, optional `systemPrompt`/`systemPromptAddition`, optional `skip` (string reason). Individual questions also support `skip` (string reason). Skipped sets/questions are logged but not executed or counted. Assertions use Ansible-style key-based format (3 types): **toolCall** (unified: tool/tools/oneOf, optional count/minCount/args), **toolCallOrder** (ordered sequence, value is direct array of steps), **answerContains** (substring or pattern). Each assertion can have an optional `name` for failure messages. Composites `allOf`/`anyOf` for logical grouping. **validDsn** is a special key inside `toolCall.args` (or `toolCallOrder` step `args`) whose value is a canonical DSN string (e.g. `USER.SRC.COBOL(CUSTFILE)`); the runner accepts any server-equivalent form (quoted, parenthesized, separate dsn+member). A tool DSN registry (`src/tool-dsn-registry.ts`) maps each tool to its dsnParam/memberParam; using `validDsn` for an unregistered tool fails fast. DSN normalization lives in `src/dsn-utils.ts`.

## Key Architectural Decisions

- **npm package name (MCP server)**: Published as **`@zowe/mcp-server`**. The CLI executable name remains **`zowe-mcp-server`** (see `bin` in the server `package.json`), so `npx @zowe/mcp-server` and the global binary `zowe-mcp-server` behave the same after install.
- **CI npm tarball**: GitHub Actions runs `npm pack -w @zowe/mcp-server` after tests pass and uploads the tarball as artifact **`zowe-mcp-server-npm`** (filename pattern `zowe-mcp-server-<version>.tgz`). Locally: `npm run pack:server` writes the same file to the repo root (ignored by `.gitignore`). The **`prepack`** script (`scripts/bundle-for-pack.cjs`) rewrites workspace deps (`zowe-mcp-common` → `.local/`) and file-based tgz deps (`zowex-sdk` → `.unpack/`), copies the rewritten `package.json` to an isolated temp directory outside the monorepo, runs `npm install --omit=dev` there (so deps are not hoisted to the workspace root), copies the resulting `node_modules` back, and adds `bundledDependencies: true` so `npm pack` includes the full tree. The **`postpack`** script restores the original `package.json` and cleans up. **Important**: `bundledDependencies` must NOT be in the committed `package.json` — it would cause `npm install` to skip deps during development (npm marks them as `inBundle` in the lockfile). The prepack script adds it dynamically. Both scripts use shared helpers from `scripts/bundle-production-deps.cjs` (also used by the VSIX `bundle-server.js`). Test airgapped installation with `npm run test:airgap` (uses empty cache, invalid registry, 5ms timeout), `npm run test:airgap:build` (builds and packs first), or `npm run test:airgap:build:native` (also runs a native z/OS smoke test using `native-config.json` and `.env` from the repo root).
- **Standalone MCP clients (e.g. Roo Code)**: Some clients only read their own MCP config (e.g. `.roo/mcp.json`) and do not use VS Code’s registered MCP providers. Use **`@zowe/mcp-server`** in stdio mode with `--native` / `--mock`, env passwords (`ZOWE_MCP_PASSWORD_*` and/or `ZOWE_MCP_CREDENTIALS` — see **Standalone env passwords** below), optional `--config` for job cards. See **`docs/roo-or-standalone-mcp.md`** and **`docs/examples/roo-mcp.json`**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zowe/zowe-mcp](https://github.com/zowe/zowe-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
