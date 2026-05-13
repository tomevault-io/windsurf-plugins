---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chrome Extension that extracts AI conversations from Google Gemini, Claude AI, ChatGPT, and Perplexity and saves them to Obsidian via the Local REST API. Built with CRXJS + Vite + TypeScript.

## ⚠️ Absolute Rules

### 🚫 NEVER (Absolutely Forbidden)

The following are **forbidden under any circumstances**. No exceptions. No context override.

#### Documentation

- **NEVER** create project-related docs under `~/.claude/`
- **NEVER** place design docs in global directories
- **NEVER** create files in `~/.claude/plans/` even in Plan mode

#### Assumptions

- **NEVER** say "should be" or "probably" without verifying server state
- **NEVER** assume current state based on past information
- **NEVER** assert "already done" without verification
- **NEVER** apply "best practices" without validation
- **NEVER** speculate on the cause of a failure — read the actual output first
- **NEVER** dismiss a failure or problem as "cascading" or "unrelated" without evidence

#### Implementation

- **NEVER** generate code before plan approval
- **NEVER** expand scope during execution phase
- **NEVER** guess configuration parameters
- **NEVER** try alternatives without error analysis
- **NEVER** modify files directly on the main branch

### ✅ ALWAYS (Mandatory Actions)

The following are **always required**. No shortcuts.

#### Verification

- **ALWAYS** follow: "I'll check" → actually check → report results
- **ALWAYS** say "verification needed" when uncertain
- **ALWAYS** read ALL details of a problem before attempting a fix
- **ALWAYS** confirm every issue independently — never assume one problem explains another
- **ALWAYS** ask the user before making any decision that has alternatives — merge strategy, file deletion, scope of changes, etc.

#### Documentation

- **ALWAYS** place project docs under `docs/`
- **ALWAYS** document reasons for config changes in comments
- **ALWAYS** create ADR for significant config changes

#### Implementation Process

- **ALWAYS** output [PLAN] before implementation
- **ALWAYS** wait for explicit approval before execution
- **ALWAYS** follow approved plan strictly
- **ALWAYS** output progress for each step
- **ALWAYS** Create a branch with an appropriate name and switch to it before making any modifications

**ADR Guide:**

- Location: `docs/adr/`
- Naming: `NNN-<topic>.md`

## Commands

Nix is the canonical task surface (see [ADR-011](docs/adr/011-nix-task-surface.md)); `npm run` continues to work as a compatibility alias. Both invoke the same `node_modules/.bin/*` binaries.

```bash
nix run .#build           # TypeScript check + Vite production build
nix run .#build-zip       # Build + zip dist/ for Chrome Web Store
nix run .#dev             # Vite dev server with HMR
nix run .#lint            # ESLint on src/ + platform consistency check
nix run .#lint-platforms  # Platform consistency check only
nix run .#format          # Prettier formatting (write)
nix run .#format-check    # Prettier formatting (check only, for CI)
nix run .#test            # Run test suite (vitest)
nix run .#test-watch      # Run tests in watch mode
nix run .#test-coverage   # Run tests with coverage report
```

Naming: Nix attribute names cannot contain `:`, so `npm run e2e:auth` maps to `nix run .#e2e-auth`. If `node_modules/` is missing, the wrapper exits with an instruction to run `npm ci` first (no auto-install per project rules).

### E2E Selector Validation

```bash
nix run .#e2e-auth                 # Manual login for all platforms
nix run .#e2e-selectors            # Run selector validation
nix run .#e2e-daemon -- start      # Start CDP daemon (headless Chrome + keep-alive)
nix run .#e2e-daemon -- stop       # Stop CDP daemon
nix run .#e2e-daemon -- status     # Check daemon health + open tabs
```

Re-authentication workflow: `e2e-daemon -- stop` → `e2e-auth` → `e2e-daemon -- start`

Load the extension in Chrome: `chrome://extensions` → Load unpacked → select `dist/` folder

## Architecture

```
Content Script (gemini.google.com, claude.ai, chatgpt.com, www.perplexity.ai, notebooklm.google.com)
    ↓ extracts conversation / Deep Research / Artifacts
Background Service Worker
    ↓ sends to Obsidian
Obsidian Local REST API (127.0.0.1:27123)
```

### Key Components

| Path                               | Purpose                                                    |
| ---------------------------------- | ---------------------------------------------------------- |
| `src/content/extractors/gemini.ts` | DOM extraction for Gemini conversations & Deep Research    |
| `src/content/extractors/claude.ts` | DOM extraction for Claude conversations & Artifacts        |
| `src/content/extractors/base.ts`   | Abstract extractor with selector fallback & title helpers  |
| `src/content/markdown.ts`          | HTML→Markdown via Turndown with custom rules               |
| `src/lib/obsidian-api.ts`          | REST API client for Obsidian                               |
| `src/lib/path-utils.ts`            | Path security & `{platform}` template resolution           |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sho7650/obsidian-AI-exporter](https://github.com/sho7650/obsidian-AI-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
