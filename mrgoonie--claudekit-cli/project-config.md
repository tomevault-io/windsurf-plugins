---
trigger: always_on
description: **This CLI is the front door to ClaudeKit.** Every command, prompt, and message serves one purpose: **empower users to understand and adopt the CK stack.**
---

# ClaudeKit CLI

## 🎯 Core Mission

**This CLI is the front door to ClaudeKit.** Every command, prompt, and message serves one purpose: **empower users to understand and adopt the CK stack.**

### The Two Imperatives

1. **Educate** — Users must understand what ClaudeKit is, what each kit offers, and why it matters to their workflow. No blind installation. Informed adoption.

2. **Install** — Zero friction from discovery to working setup. Whether Engineer, Marketing, or both — the path must be clear, fast, and successful.

### Design Philosophy

- **Show, don't tell** — Feature previews over marketing copy
- **Guide, don't gatekeep** — Sensible defaults, optional depth
- **Succeed, don't abandon** — Every install ends with working config + clear next steps
- **Respect time** — Fast paths for experts, guided paths for newcomers

### The Kits

| Kit | Purpose | Audience |
|-----|---------|----------|
| **Engineer** | AI-powered coding: skills, hooks, multi-agent workflows | Developers building with Claude |
| **Marketing** | Content automation: campaigns, social, analytics | Marketers leveraging AI |

Both kits share the ClaudeKit foundation. Users can install one or both.

---

CLI tool (`ck`) for bootstrapping/updating ClaudeKit projects from GitHub releases.

## 🎯 Core Principle

**User experience is paramount.** The CLI is users' first touchpoint with ClaudeKit. Prioritize clarity over cleverness: intuitive commands, helpful errors, minimal friction from install to daily use.

---

## CRITICAL: Quality Gate

**MUST pass before ANY commit/PR. No exceptions.**

```bash
bun run validate
# Equivalent to: bun run typecheck && bun run lint && bun test && bun run build
# Note: validate uses lint (read-only check), not lint:fix. Run lint:fix manually first.
```

**When touching UI files (`src/ui/`):** The pre-push hook runs `bun run ui:build` automatically. The UI has a stricter TypeScript config (`tsc -b`) that catches errors `tsc --noEmit` misses (unused vars, missing ES lib methods like `Array.at()`). If debugging CI failures manually, run `bun run ui:build` from root.

**Enforced by git hooks** — `pre-commit` runs typecheck+lint+build, `pre-push` adds tests. Hooks auto-install on `bun install`. If hooks are missing, run `bun run install:hooks`.

**AI agents: NEVER use `--no-verify` to bypass hooks. NEVER set `SKIP_HOOKS=true`. If the hook rejects your commit, fix the code — do not skip the gate. This rule is NON-NEGOTIABLE.**

**Human bypass (emergencies only):** `SKIP_HOOKS=true git commit -m "..."` or `SKIP_HOOKS=true git push`.

**Why:** AI-generated code historically failed CI in 80%+ of PRs, causing 3-6 fix-up commits each. The hooks exist to catch these failures locally before they waste CI cycles and pollute git history.

**Worktree support:** Hooks work in both normal repos and worktrees. The install script uses `core.hooksPath` with a relative path (`.githooks`) that resolves from each worktree's root. After creating a worktree, run `bun install` or `bun run install:hooks` from the worktree root.

**Common pitfalls:**
- Web server deps (`express`, `ws`, `chokidar`, `get-port`, `open`) must be in `package.json` — not just transitive
- UI component files must pass biome formatting (long JSX lines auto-wrapped)
- Express 5 types `req.params`/`req.query` as `string | string[]` — cast with `String()`

## Quick Commands

```bash
# Development
bun install                    # Install deps
bun run dev new --kit engineer # Run locally
bun test                       # Run tests
bun run lint:fix               # Auto-fix lint
bun run typecheck              # Type check
bun run build                  # Build for npm
bun run dashboard:dev          # Start config UI dashboard

# Testing
bun test <file>                # Single file
bun test --watch               # Watch mode
```

## Dashboard Development (Config UI)

```bash
bun run dashboard:dev     # Start dashboard (Express+Vite on :3456)
```

- **Single port:** http://localhost:3456 (auto-fallback 3456-3460)
- Backend API + Vite HMR served together
- **DO NOT** use `cd src/ui && bun dev` alone — no API backend, everything breaks
- Source: `src/commands/config/config-ui-command.ts` → `src/domains/web-server/`

## Desktop App (Tauri v2)

ClaudeKit ships a native desktop app ("Control Center") built with Tauri v2 (Rust backend + React frontend).

### Architecture

The dashboard React app (`src/ui/`) runs in two modes:
- **Web mode** — served via `ck config ui` (Express + Vite on :3456)
- **Desktop mode** — embedded in Tauri webview, detected by `isTauri()` from `src/ui/src/hooks/use-tauri.ts`

The Rust backend (`src-tauri/`) provides native capabilities (filesystem, tray, auto-update) via Tauri commands. The frontend calls these via `@tauri-apps/api`.

### Rust Backend Structure

```
src-tauri/
├── tauri.conf.json     # App config (build, CSP, updater, icons)
├── Cargo.toml          # Rust dependencies
├── capabilities/       # Permission grants (store, dialog, updater)
├── icons/              # App icons (generated from src/ui/public/images/logo-512.png)
└── src/
    ├── lib.rs          # Tauri builder: plugins, setup, command registration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrgoonie/claudekit-cli](https://github.com/mrgoonie/claudekit-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
