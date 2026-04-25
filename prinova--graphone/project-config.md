---
trigger: always_on
description: - **Check local code first** - Before using GitHub tools (`read_github`, `search_github`, etc.), always explore the local codebase using standard file tools (`read`, `bash` with `find`/`grep`, etc.)
---

# Graphone - Agent Quick Reference

## Agent Workflow Guidelines

- **Check local code first** - Before using GitHub tools (`read_github`, `search_github`, etc.), always explore the local codebase using standard file tools (`read`, `bash` with `find`/`grep`, etc.)
- **Check pi-mono first for feature implementations** - Since graphone is a visual wrapper around pi-mono, look up implementations in local `../pi-mono` first (when available). Many features already exist there and can be mimicked or adapted for desktop UI
- Use GitHub tools only when you need to reference external repositories or when explicitly asked about remote code
- **Do not track or push `docs/` changes** - The `docs/` folder and all files under it are local-only in this repo context. Do not `git add`/commit/push anything in `docs/` (including with `git add -f`) unless the user explicitly overrides this rule in the current task.
- **Use title + body commit messages by default** - Unless the user explicitly requests otherwise, write commits with a concise title and a descriptive body (not title-only).
- **Use robust multiline commit messages** - Do not embed literal `\n` in a single `git commit -m` string. Use multiple `-m` flags (one per paragraph) or `git commit -F` with a heredoc/file so line breaks render correctly on GitHub.
- **PAT push hygiene** - If pushing with `GITHUB_PAT`, pass it via git auth headers/env only; never echo, print, or otherwise expose the token in commands/output.
- **Default autonomous iteration guidance** - For general coding-agent-first execution patterns (tests, TDD/debug loops, logs, traces, artifacts, contracts, smokes, escalation), see `docs/agent-first-autonomous-iteration-playbook.md`
- **Visual verification uses readiness loops, not fixed sleeps** - Follow the global workflow `bash -> readiness loop -> take_screenshot -> bash cleanup`. The readiness wait must happen inside the startup `bash` call because screenshot tools cannot run until that tool call returns.
- **Do not start and stop dev helpers in parallel** - Startup and cleanup are separate sequential tool calls, with `take_screenshot` between them. Running `start-dev-and-wait.sh` and `stop-dev.sh` in parallel is invalid and defeats the workflow.
- **Prefer project helper scripts for visual checks** - For Graphone, use `tooling/scripts/start-dev-and-wait.sh` and `tooling/scripts/stop-dev.sh` instead of rewriting startup/cleanup loops inline.

## Visual Verification Hints (Graphone)

- Use the project helper scripts instead of writing long inline readiness loops in `bash` commands.
- Preferred sequence for visual checks:
  1. `bash tooling/scripts/start-dev-and-wait.sh`
  2. `take_screenshot`
  3. `bash tooling/scripts/stop-dev.sh`
- Never run steps 1 and 3 in parallel.
- Startup helper artifacts:
  - log: `/tmp/graphone-dev.log`
  - process group: `/tmp/graphone-dev.pgid`
  - app pid: `/tmp/graphone-dev.app.pid`
- `tooling/scripts/start-dev-and-wait.sh` waits for these Graphone readiness signals:
  1. Vite log contains `VITE` and `ready` (or the local URL line)
  2. Tauri has launched `target/debug/graphone`
  3. The debug desktop process exists
- Avoid taking screenshots while only Vite is ready but the Tauri desktop window has not launched yet
- If startup fails or times out, inspect `/tmp/graphone-dev.log`

## Build Commands

- `npm install` - Install dependencies
- `npm run check` - Type check Svelte/TypeScript (runs automatically on build)
- `npm run check:watch` - Type check in watch mode (for development)
- `npm run check:repo` - Run repository guardrails (legacy path/symlink regression checks)
- `npm run format` - Format all code (Svelte, TypeScript, Markdown, Rust)
- `npm run format:check` - Check if code is formatted (CI-style, exits with error if not)
- `npm run dev:linux` - Dev server (Linux native)
- `npm run dev:windows` - Dev server (Windows cross-compile)
- `npm run build` - Type check + build frontend (runs `check` first)
- `npm run build:linux` - Full Linux AppImage/deb build (includes type check)
- `npm run build:windows` - Full Windows NSIS installer build (includes type check)
- `npm run build:windows:exe` - Build Windows .exe only (no installer needed)
- `npm run build:windows:portable` - Build Windows .exe + stage portable runtime folder with sidecar assets
- `npm run build:macos:local` - Build local ad-hoc signed macOS `.app` + `.dmg`
- `npm run build:macos:local:app` - Build local ad-hoc signed macOS `.app` only
- `npm run build:all` - Build Linux + Windows
- `npm run run:windows` - Build (if needed), stage portable runtime, and launch the Windows app via host interop (if available)

## Stack & Architecture

- **Frontend**: Svelte 5 + TypeScript + Vite
- **Desktop shell**: Rust + Tauri 2.0 (`src-tauri`) for native desktop packaging/windowing
- **Canonical runtime/service**: Graphone-local SDK host sidecar (`services/agent-host`, compiled with bun)
- **Pattern**: Desktop uses one host sidecar process that multiplexes multiple in-process agent sessions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PriNova/graphone](https://github.com/PriNova/graphone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
