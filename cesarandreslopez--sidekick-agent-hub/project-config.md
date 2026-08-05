---
trigger: always_on
description: This file provides guidance to Codex and other coding agents when working with code in this repository. `CLAUDE.md` mirrors the same project guidance for Claude Code.
---

# AGENTS.md

This file provides guidance to Codex and other coding agents when working with code in this repository. `CLAUDE.md` mirrors the same project guidance for Claude Code.

## Project Overview

Sidekick Agent Hub is an AI coding assistant with real-time agent monitoring. It ships as a VS Code extension and a terminal dashboard, using Claude Max, Claude API, OpenCode, or Codex CLI for inference and session monitoring.

The repo is a small monorepo:

- `sidekick-vscode/` — VS Code extension, extension-host services, and webview source
- `sidekick-shared/` — shared TypeScript library used by the extension and CLI; published as `sidekick-shared`
- `sidekick-cli/` — Ink-based terminal dashboard; published as `sidekick-agent-hub` with the `sidekick` binary
- `docs/`, `mkdocs.yml`, `assets/`, `images/` — documentation site content and assets
- `scripts/` — cross-package build, lint, and version helpers

## Build & Development Commands

Extension commands run from `sidekick-vscode/`:

```bash
npm run compile      # Dev build with source maps (esbuild)
npm run build        # Production build, minified
npm run watch        # Watch mode for development
npm test             # Run all tests (Vitest)
npm run test:watch   # Watch mode for tests
npm run lint         # ESLint check
npm run lint:fix     # ESLint auto-fix
npm run format       # Prettier write for this package
npm run format:check # Prettier check for this package
npm run package      # Create .vsix for distribution
```

Run a single test file: `npx vitest run src/services/ModelResolver.test.ts` (from `sidekick-vscode/`).

Press **F5** in VS Code with `sidekick-vscode/` open to launch the Extension Development Host.

Shared library commands run from `sidekick-shared/`:

```bash
npm run build        # tsc build to dist/
npm test             # Build, then run Vitest
npm run lint         # ESLint check
npm run format       # Prettier write for this package
npm run format:check # Prettier check for this package
```

CLI commands run from `sidekick-cli/`:

```bash
npm run build        # esbuild ESM binary to dist/sidekick-cli.mjs
npm test             # Run Vitest
npm run lint         # ESLint check
npm run format       # Prettier write for this package
npm run format:check # Prettier check for this package
```

**Monorepo-wide helpers** (run from repo root) cover all three packages — `sidekick-shared`, `sidekick-vscode`, `sidekick-cli`:

```bash
bash scripts/lint-all.sh          # Lint all three packages (CI lints each separately)
bash scripts/lint-all.sh --fix    # Lint + auto-fix all three
bash scripts/format-all.sh        # Prettier write across packages, docs, root markdown/YAML, and workflows
bash scripts/format-check-all.sh  # Prettier check across packages, docs, root markdown/YAML, and workflows
bash scripts/build-all.sh         # npm install + build all three; CLI binary at sidekick-cli/dist/sidekick-cli.mjs
bash scripts/bump-version.sh X.Y.Z # Update package.json versions; sync lockfiles separately
```

### Documentation Site

The docs site uses **zensical** (not mkdocs). Config is in `mkdocs.yml` at the repo root, content in `docs/`.

```bash
zensical build --strict   # Build docs site (from repo root)
zensical serve            # Local dev server with hot reload
```

Do **not** use `mkdocs build` or `mkdocs serve` — use `zensical` instead.

## Architecture

### Build System (esbuild.js)

`sidekick-vscode/esbuild.js` produces five bundles:

| Output                                       | Format   | Platform |
| -------------------------------------------- | -------- | -------- |
| `out/extension.js` (from `src/extension.ts`) | CommonJS | Node.js  |
| `out/webview/explain.js`                     | IIFE     | Browser  |
| `out/webview/error.js`                       | IIFE     | Browser  |
| `out/webview/chartjs-vendor.js`              | IIFE     | Browser  |
| `out/webview/d3-vendor.js`                   | IIFE     | Browser  |

Only `vscode` is externalized from the extension-host bundle. Other extension dependencies (including `@anthropic-ai/claude-agent-sdk`, `@opencode-ai/sdk`, and `sidekick-shared`) are bundled by esbuild. The `conditions: ['import']`, `banner`, and `define` settings in `esbuild.js` polyfill `import.meta.url` for ESM deps bundled into CJS. Chart.js and D3.js are bundled into local browser vendor files so the dashboard and mind map work offline.

### Dual Provider System

Two separate provider concepts exist:

1. **Inference providers** (`InferenceProviderId` in `src/types/inferenceProvider.ts`): `claude-max | claude-api | opencode | codex` — which service generates AI completions
2. **Session providers** (`SessionProvider` in `src/types/sessionProvider.ts`): `claude-code | opencode | codex` — which CLI agent's sessions to monitor

Both use auto-detection via `ProviderDetector` based on filesystem presence and most-recent mtime.

### ClaudeClient Interface

All inference clients implement `ClaudeClient` from `src/types.ts`:

```typescript
interface ClaudeClient {
  complete(prompt: string, options?: CompletionOptions): Promise<string>;
  isAvailable(): Promise<boolean>;
  dispose(): void;
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cesarandreslopez/sidekick-agent-hub](https://github.com/cesarandreslopez/sidekick-agent-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
