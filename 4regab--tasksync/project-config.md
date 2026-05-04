---
trigger: always_on
description: TaskSync is a human-in-the-loop workflow toolkit for AI-assisted development. The primary codebase is the VS Code extension in `tasksync-chat/`. It provides a sidebar with smart prompt queuing, Autopilot mode, and remote access via WebSocket.
---

# TaskSync — Copilot Repository Instructions

## What This Repository Does

TaskSync is a human-in-the-loop workflow toolkit for AI-assisted development. The primary codebase is the VS Code extension in `tasksync-chat/`. It provides a sidebar with smart prompt queuing, Autopilot mode, and remote access via WebSocket.

## Project Layout

- `tasksync-chat/src/extension.ts` — Extension entry point
- `tasksync-chat/src/tools.ts` — VS Code language model tool definitions (`ask_user`)
- `tasksync-chat/src/webview/webviewProvider.ts` — Orchestrator: owns state, creates webview, delegates to handlers
- `tasksync-chat/src/webview/webviewTypes.ts` — Shared types (`P` interface, message unions)
- `tasksync-chat/src/webview/webviewUtils.ts` — Shared helpers (`debugLog()`, `mergeAndDedup()`, `notifyQueueChanged()`)
- `tasksync-chat/src/webview/*Handlers.ts` — Handler modules receive a `P` interface (no circular imports)
- `tasksync-chat/src/server/` — Remote server, auth, git operations, HTML service
- `tasksync-chat/src/constants/` — Shared constants (config keys, file exclusions, remote constants)
- `tasksync-chat/src/context/` — Context providers (files, terminal, problems)
- `tasksync-chat/src/utils/` — Utilities (ID generation, image handling)
- `tasksync-chat/esbuild.js` — Build script (extension, webview, shared constants, mermaid)
- `tasksync-chat/biome.json` — Biome linter/formatter config
- `tasksync-chat/vitest.config.ts` — Test config
- `tasksync-chat/web/` — Remote access PWA (login, service worker)
- `Prompt/` — Standalone prompt/protocol markdown files (not actively developed)

## Build, Test, and Validate

All commands run from `tasksync-chat/`:

```bash
cd tasksync-chat
npm install          # Always run first
npm run validate     # Full validation: build + tsc + test + lint + code quality scanner
```

Or run individually:

```bash
node esbuild.js      # Build → dist/extension.js, media/webview.js, web/shared-constants.js
npx tsc --noEmit     # Type-check (must produce 0 errors)
npx vitest run       # Run all tests (387+ tests, must all pass)
npm run lint         # Biome lint (must produce 0 issues)
npm run check-code        # Code quality scanner (duplicates, sync I/O, etc.)
```

Always run these four checks (build, tsc, vitest, lint) after making changes. The CI workflow (`.github/workflows/auto-release.yml`) runs on pushes to `main` — it installs deps, builds, version-bumps, packages VSIX, and creates a GitHub release.

Build output: `dist/` (extension bundle), `media/webview.js` (webview bundle), `web/shared-constants.js` (auto-generated for remote PWA).

## Code Style and Conventions

- **TypeScript** with `"strict": true`, ES2022 target, CommonJS modules
- **Indentation:** Tabs (enforced by Biome)
- **Quotes:** Double quotes (enforced by Biome)
- **Imports:** Auto-organized by Biome (`organizeImports: on`)
- **Type assertions:** Use `satisfies` over `as` (e.g., `} satisfies ToWebviewMessage`). `satisfies` validates shape at compile time; `as` silently bypasses checks.
- **Async I/O:** Always prefer async file operations over synchronous equivalents. Never use synchronous blocking calls on the extension host.
- **Logging:** Use `debugLog()` from `webviewUtils.ts` (gated behind `tasksync.debugLogging` config). Use `console.error` only for genuine errors. Never use `console.log` or `console.warn` in production code.
- **Changelog:** Update `CHANGELOG.md` for every user-facing change using the format `## TaskSync vX.Y.Z (MM-DD-YY)` (two-digit year).

## SSOT / DRY / KISS / YAGNI Principles

These principles are mandatory for all changes:

- **Single Source of Truth (SSOT):** Every concept, constant, type, or piece of logic must have exactly one canonical definition. Do not duplicate config keys, message types, validation logic, or shared helpers. Constants live in `src/constants/`. Shared types live in `webviewTypes.ts`. Shared helpers live in `webviewUtils.ts`.
- **Don't Repeat Yourself (DRY):** If logic is used in more than one place, extract it into a shared helper. Examples already in the codebase: `debugLog()`, `mergeAndDedup()`, `notifyQueueChanged()`, `hasQueuedItems()`, `resolveFilePath()` (gitService). When you see the same pattern in 3+ call sites, extract it.
- **Keep It Simple, Stupid (KISS):** Prefer the simplest solution that works. Do not add abstraction layers, configuration options, or indirection without clear justification. A small amount of duplication is acceptable if the alternative is a complex abstraction for only 2 call sites.
- **You Aren't Gonna Need It (YAGNI):** Do not add features, parameters, or code paths "just in case." Only implement what is needed for the current task. Do not design for hypothetical future requirements.
- **Handler pattern:** All handler modules (`*Handlers.ts`) receive a `P` interface — do not add direct imports from `webviewProvider.ts`. This prevents circular dependencies.

## Security Best Practices

Follow OWASP Top 10 principles. Specific patterns enforced in this codebase:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [4regab/TaskSync](https://github.com/4regab/TaskSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
