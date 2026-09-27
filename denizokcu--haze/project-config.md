---
trigger: always_on
description: Last updated: 2026-08-31 for the 1.2.0 release.
---

# AGENTS.md

Last updated: 2026-08-31 for the 1.2.0 release.

Project instructions for haze coding agents. Keep this root file concise; read nested `AGENTS.md` files in the subtree you touch for precise contracts.

Last analysis: 2026-08-15.

## Project overview

haze is a Node >=22 TypeScript ESM CLI package (`@denizokcu/haze`) for terminal-based agentic app building.

Core shape:

- React + Ink interactive terminal chat UI, themed through a built-in palette registry (`/themes`).
- Vercel AI SDK with OpenAI-compatible providers.
- Local tools for file discovery/read/search/edit/write, public URL fetch, foreground and managed background processes, LSP/MCP integration, global/project skills, image attachments, subagents/fleet, task tracking, session browsing/forking, and compaction.
- Source lives in `src/`; generated `dist/` must not be edited.

Verify current package version in `package.json` before release work.

## Common commands

```bash
npm install
npm ci                 # preferred in CI or clean checkouts
npm run dev            # run CLI via tsx
npm run haze           # alias for dev
npm start              # run built dist CLI

npm run typecheck      # tsc --noEmit
npm test               # vitest run
npm run lint           # eslint src/
npm run context:report # estimated prompt/tool/context token breakdown

npm run build          # clean + tsc
npm pack --dry-run     # inspect published tarball
```

Before release/PR confidence: `npm run typecheck && npm test && npm run lint && npm run build`.

## Repository map

- `src/` — TypeScript/TSX source. See nested `src/**/AGENTS.md` files.
- `tests/` — Vitest suite. See `tests/AGENTS.md`.
- `bin/haze.js` — thin npm binary shim to built CLI.
- `dist/` — generated build output; never edit directly.
- `docs/*.html` — static docs pages in repo (`index.html` plus topic pages like `commands.html`, `tools.html`).

## Global coding conventions

- Strict TypeScript, ESM (`type: "module"`), NodeNext module resolution, ES2022 target.
- Local TypeScript imports use `.js` extensions.
- Prefer plain TypeScript for core logic; keep React/Ink in CLI/UI layers.
- Use Zod for AI SDK tool schemas and generated-object schemas.
- YAML parsing/writing uses the `yaml` package.
- Avoid `any`; prefer `unknown`, type guards, or existing result types.
- Preserve local formatting style; avoid broad formatting churn.
- ESLint: unused vars are errors unless args start with `_`; `no-explicit-any` is an error.

## Editing rules

- Check `git status --short` before large work; do not overwrite unrelated user edits.
- Never edit `dist/`, `node_modules/`, `.git/`, generated outputs, secrets, or ignored runtime state.
- Do not edit `package-lock.json` unless dependency changes require it.
- Prefer targeted edits over whole-file rewrites for source.
- Do not commit, tag, publish, reset, delete, force-push, or run destructive cleanups unless explicitly requested.

## Runtime contracts to preserve

Recent decisions to preserve:

- Runtime support floor is Node >=22. Keep docs, package metadata, and generated docs aligned.
- Provider/model selection is explicit: do not silently fall back to the first configured provider/model.
- Settings parsing should fail loudly for malformed files and preserve unrelated/unknown fields when patching.

- No default provider/model. Users configure providers via `/provider`; no user-facing env vars for provider/model settings.
- The provider surface stays OpenAI-compatible (plus the `chatgpt-codex` OAuth kind) for now; frontier models are reached through gateways (OpenRouter, Gemini's official OpenAI endpoint). Do not add native provider adapters without an explicit decision change — the `kind` union on `HazeProviderSettings` is the extension point.
- File tools are confined to `process.cwd()`, respect `.gitignore` by default, and skip `.git`/`node_modules` walking; user-typed `@path` mentions and bare paths containing `/` are the exception and may bless host paths outside it for read-only tools (readFile, grep, listFiles). Mutating tools never honour the bless set. URL safety fails closed for malformed IPv6-shaped literals.
- Secret files are hands-off for the file tools, always: SSH keys (`~/.ssh/**`, `id_*` private keys), shell history files, `.env`/`.envrc` (except `.example`/`.sample`/`.template` doc variants), `*.pem`/`*.key`, and common home credential stores (`core/safety/secretPaths.ts`). Reads and mutations are refused before any filesystem access with a terminal structured result (reason code `secret_file_protected`, `recoverable: false`, ask-the-user next step, no content echoed), the check covers lexical and real paths so symlinks cannot rename secrets into reach, and it overrides the bless set and `allowIgnored`; grep traversal excludes the names with negated-only globs appended after any model glob. The UI summarizes these as `blocked: protected secret file (path)`, distinct from ordinary failures. The shell tool is deliberately not hard-filtered; shell-side secret avoidance is instructed via the system prompt (`SECRET_FILE_RULE`), so the prompt rule and the file-tool refusal must stay aligned when either changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DenizOkcu/haze](https://github.com/DenizOkcu/haze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
