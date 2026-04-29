---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
bun install          # NOT npm/yarn — this project uses Bun exclusively
bun run build        # bundles to dist/cli.js (~23MB single file)
bun dist/cli.js      # run the CLI
```

There are no tests. There is no linter. The build script is `build.ts` (not a bundler config file).

## What This Repo Is

Rebuilt Claude Code CLI from Anthropic's leaked source (1,929 files extracted from a source map). The `src/` directory is the original source. The `stubs/` directory contains proprietary code extracted from npm packages and source maps. This is not a fork — it's a reconstruction.

## Critical Gotchas

- **Feature flags:** All `feature('FLAG_NAME')` calls return `false` via the shim at `shims/bun-bundle.ts`. This is intentional — it disables unreleased/internal features. Do not change this.
- **MACRO.VERSION** is hardcoded to `2.1.88` in `build.ts`. Do not change this.
- **Auto-updater is patched:** `src/utils/autoUpdater.ts` line 72 has an early return to disable remote version checks. Do not remove this.
- **[STUB] files** contain minimal placeholder code. They exist because the source was missing (not in the leak), requires native binaries (Swift/Rust), or is Anthropic-internal (`USER_TYPE === 'ant'`). Do not try to "fix" or "complete" stubs unless explicitly asked.
- **stubs/ files are extracted source** — treat them as read-only reference material, not code to modify.

## Architecture (non-obvious parts only)

- **Entrypoint:** `src/main.tsx` — Commander.js CLI parser. Bootstrap is `src/entrypoints/cli.tsx`.
- **UI:** Custom Ink fork at `src/ink/` — this is NOT the npm `ink` package. It's a full React-based terminal renderer with custom flexbox layout (`src/native-ts/yoga-layout/`).
- **System prompt:** `src/constants/prompts.ts` — the actual instructions sent to Claude. This is the single most important file for understanding Claude Code's behavior.
- **Tools** live in `src/tools/` with each tool in its own directory (BashTool/, FileEditTool/, etc.).
- **Permissions classifier:** `src/utils/permissions/yoloClassifier.ts` (52KB) — auto-mode logic. The 3 prompt `.txt` files it references were dead-code-eliminated and are missing.
- **Plugin system:** `src/utils/plugins/` (65+ files) — fully working. Loads from `.claude/plugins/`.
- **Hook system:** `src/utils/hooks/` (155 files) — fully working. Config in `settings.json`.
- **CLAUDE.md loader:** `src/utils/claudemd.ts` — how this very file gets loaded. Supports `@path` includes, frontmatter with `paths:` globs, HTML comment stripping.

## Backported Changes (2.1.89 → 2.1.91)

The base source is v2.1.88. The following features and fixes from the official 2.1.89–2.1.91 changelogs have been backported:

### From 2.1.91
- **`disableSkillShellExecution` setting** — disables `!`cmd`` and ````!``` shell execution in skills/commands/plugins (`types.ts`, `promptShellExecution.ts`)
- **MCP `_meta["anthropic/maxResultSizeChars"]` override** — MCP tools can request up to 500K chars for large results like DB schemas (`mcpValidation.ts`, `client.ts`)
- **Multi-line prompts in deep links** — encoded newlines `%0A` no longer rejected in `claude-cli://open?q=` (`parseDeepLink.ts`)
- **Plugin `bin/` executables** — plugins can ship executables under `bin/` that are prepended to PATH for Bash tool (`plugin.ts`, `pluginLoader.ts`, `subprocessEnv.ts`)
- **Faster `stripAnsi` via `Bun.stripANSI`** — native Bun path when available, fallback to `strip-ansi` package (`src/utils/stripAnsi.ts` + 12 import sites)

### From 2.1.90
- **`.husky` added to protected directories** — prevents auto-editing in acceptEdits mode (`filesystem.ts`)
- **DNS commands removed from auto-allow** — `Get-DnsClientCache` fully removed, `ipconfig /displaydns` removed for DNS cache privacy (`readOnlyValidation.ts`)
- **`CLAUDE_CODE_PLUGIN_KEEP_MARKETPLACE_ON_FAILURE` env var** — keeps existing marketplace cache when git pull fails, useful in offline environments (`marketplaceManager.ts`)

### From 2.1.89
- **`cleanupPeriodDays: 0` validation** — now rejected with actionable error pointing to `--no-session-persistence` (`types.ts`)
- **`defer` permission decision for PreToolUse hooks** — headless sessions can pause and resume via `--resume` (`PermissionRule.ts`, `coreSchemas.ts`, `hooks.ts`)
- **`MCP_CONNECTION_NONBLOCKING=true` env var** — bounds MCP server connections at 5s instead of 30s (`client.ts`)
- **Autocompact thrash loop** — actionable error message when circuit breaker trips after 3 consecutive failures (`autoCompact.ts`)
- **MCP tool errors: all content blocks** — multi-element error content no longer truncated to first block (`client.ts`)
- **Edit works on files viewed via Bash** — `cat`, `sed -n`, `head`, `tail` output tracked in `readFileState` so Edit doesn't require separate Read (`BashTool.tsx`)
- **Hook output >50K saved to disk** — large hook output written to temp file with path + preview instead of injecting into context (`hooks.ts`)

### Already present in base source (no changes needed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fazxes/Claude-code](https://github.com/fazxes/Claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
