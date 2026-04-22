---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Development
npm run tauri:dev      # Primary dev server with hot reload
npm run dev            # Vite frontend only
npm run preview        # Preview production build

# Code Quality - ALWAYS RUN BEFORE COMMITTING
npm run type-check     # TypeScript validation
npm run lint           # ESLint check (CI fails on warnings)
npm run lint:fix       # Auto-fix ESLint issues
cd src-tauri && cargo check && cargo clippy  # Rust checks
cd src-tauri && cargo fmt  # Auto-fix Rust formatting

# Building
npm run tauri:build    # Creates platform installers
npm run build          # Frontend build only

# Testing
npm test               # Run Vitest in watch mode
npm run test:run       # Run tests once

# Release Management
npm run release:patch  # 0.1.0 → 0.1.1 with git tag
npm run release:minor  # 0.1.0 → 0.2.0 with git tag
npm run release:major  # 0.1.0 → 1.0.0 with git tag
```

## Pre-Commit Gate

Do not commit or ask for review until the relevant local checks pass.
Do not push if you made additional edits after the last successful run; rerun the relevant checks first.

- Always run after code changes:

  ```bash
  npm run type-check
  npm run lint
  cd src-tauri && cargo fmt --check && cargo clippy -- -D warnings
  ```

- If you touched `src/App.tsx`, file watcher handling, reload toasts, or integration flows, also run:

  ```bash
  npx vitest run tests/app.integration.spec.tsx
  ```

- If you touched MCP server, Rust backend settings, or path-resolution logic, also run:

  ```bash
  cargo test --manifest-path src-tauri/mcp-server/Cargo.toml --test gtdspace_mcp_stdio -- --nocapture
  cargo test --manifest-path src-tauri/Cargo.toml parse_user_settings_value_accepts_partial_saved_settings -- --nocapture
  ```

- If you touched frontend settings validation or MCP settings UI, also run:

  ```bash
  npx vitest run tests/settings-validation.spec.ts
  ```

- If you touched dashboard actions, overview, or projects, also run:

  ```bash
  npx vitest run tests/dashboard-actions.component.spec.tsx tests/dashboard-projects.component.spec.tsx tests/date-formatting.spec.ts
  ```

- If `cargo fmt` changes files, rerun the affected checks before committing.
- Treat CI-only failures caused by missing local verification as avoidable regressions.

## Setup Requirements

- **Node.js**: v20+ required
- **Rust**: Latest stable via [rustup](https://rustup.rs/)
- **First run**: `npm install` then `npm run tauri:dev`
- **Google Calendar**: Configure OAuth in Settings UI (or `.env` with `GOOGLE_CALENDAR_CLIENT_ID` and `GOOGLE_CALENDAR_CLIENT_SECRET`)

### Claude AI Settings Configuration

If you're using Claude Code (claude.ai/code) with this repository, you'll need to configure local settings:

1. **Copy the example file**:
   ```bash
   cp .claude/settings.local.json.example .claude/settings.local.json
   ```

2. **Edit `.claude/settings.local.json`** and replace the placeholder paths:
   - Replace `/path/to/your/workspace/**` with your actual workspace absolute path (e.g., `/Users/yourname/Desktop/code/gtdspace/**`)
   - If you need access to other directories, add additional `Read()` entries with specific paths

3. **Permissions are scoped to essential commands only**:
   - Development: `npm run tauri:dev`, `npm run dev`, `npm run build`
   - Code quality: `npm run type-check`, `npm run lint`, `cargo check`, `cargo clippy`, `cargo fmt`
   - Testing: `npm test`, `npm run test:run`
   - Git operations: `git add`, `git commit` (for PR work)
   - Utilities: `find`, `grep`, `cat`, `ls`, `rg`, `jq`, `diff`, `sed`, `rm`, `mkdir`

4. **Important**: 
   - `.claude/settings.local.json` is gitignored and contains machine-specific paths
   - Never commit your local settings file
   - The example file (`.claude/settings.local.json.example`) is sanitized and safe to commit
   - If you need additional permissions, add them to your local file only, not the example

## Architecture Overview

**Stack**: React 18 + TypeScript + Vite + Tauri 2.x (Rust backend) + BlockNote editor v0.37 (pinned)
**State Management**: Custom React hooks only - no Redux/Zustand
**Entry Points**: `src/App.tsx` (frontend), `src-tauri/src/lib.rs` (backend commands)

### Project Structure

```
src/
├── components/     # React UI (editor/, sidebar/, calendar/, dashboard/)
├── hooks/          # Business logic and state management
├── utils/          # Helpers (metadata-extractor.ts, date-formatting.ts, etc.)
├── lib/            # Shared utilities and configurations
src-tauri/
├── src/commands/   # Rust IPC commands
├── src/lib.rs      # Command registration (NOT main.rs)
scripts/            # Build automation (bump-version.js, safe-release.js)
```

### GTD Workspace Structure

Auto-created at `~/GTD Space` on first run:
- **Projects/** - Folders containing README.md + action files (.md)
- **Habits/** - Recurring tasks with frequency-based auto-reset
- **Purpose & Principles/**, **Vision/**, **Goals/**, **Areas of Focus/** - GTD horizons
- **Cabinet/**, **Someday Maybe/** - Reference and future ideas

## Core React Hooks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AustinKelsay) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
