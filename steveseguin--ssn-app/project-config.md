---
trigger: always_on
description: Electron desktop application for aggregating social media live stream chat. CommonJS JavaScript codebase (no TypeScript).
---

# AGENTS.md - Social Stream Ninja Standalone (ssapp)

## Project Overview

Electron desktop application for aggregating social media live stream chat. CommonJS JavaScript codebase (no TypeScript).

## Git Rules (CRITICAL)

- **Never create git branches, worktrees, or pull requests in this repo unless Steve explicitly asks for one.** All work happens as commits directly on the currently checked-out branch (normally `main`) in the main checkout at `C:\Users\steve\Code\ssapp`.
- This applies to every agent and tool session (Claude, Codex, or anything else). Agent-created branches and worktrees keep getting orphaned and their work lost — e.g. the May 2026 `codex/*` branches and the stale `ssapp-vpzone-fix` worktree that had to be cleaned up in August 2026.
- If a task genuinely seems to require a branch or PR, stop and ask Steve first instead of creating one.

## Communication

- When replying to Steve, prefer plain, everyday language over jargon.
- Keep explanations direct and practical; explain technical terms briefly when they matter.
- When Steve asks for a TLDR, keep it genuinely short: a few lines max, no long explanation.
- Always end a substantial reply with concrete next steps when there is a sensible one to offer: what is blocked on Steve, what is blocked on someone else, the one thing worth doing first, and an offer to start it. Include anything deferred along the way instead of quietly dropping it. Skip this only when the task is genuinely finished or the reply is a one-line factual answer.
- When Steve asks to remember an instruction, save it into the relevant instruction file or memory mechanism when possible; do not merely say it will be kept in mind.
- If Steve says "remember", treat it as a request to persist the instruction. Check for writable instruction targets, especially the repo `AGENTS.md` for project-specific behavior and `C:\Users\steve\.codex\AGENTS.md` for global behavior. Update the most appropriate file, or both when the instruction applies globally and to the current repo. Do not say memory tools are unavailable unless no writable instruction or memory target exists after checking.

## Platform Fix Scope (CRITICAL)

- Fixes for one source must affect only that source. Prefer its existing settings in `C:\Users\steve\Code\social_stream\settings\config*.json`; when those cannot express the fix, use source-specific code with explicit platform/window/request guards.
- Never remove or alter `app.commandLine.appendSwitch('--disable-web-security', 'false')` without Steve's explicit approval for that exact change. It is an intentional compatibility setting. A site working after its removal is not evidence that the rest of SSApp remains compatible.
- Do not change app-wide Electron flags, security defaults, preload behavior, headers, sessions, or shared navigation behavior to fix one source without Steve approving that broader scope first. This includes temporary diagnostic edits in the main checkout.
- Before editing, identify the full behavioral scope. A change in `main.js` is acceptable when narrowly gated; a global behavior change is not authorized by a platform-specific bug report.
- Before finishing, review the diff and verify that unrelated sources, windows, and requests retain their existing behavior. Passing a source test or general capture test does not establish that other platforms' authentication flows are unaffected.

## Source Of Truth

- Social Stream source edits must be made in `C:\Users\steve\Code\social_stream`.
- `ssapp` loads Social Stream source files remotely from `C:\Users\steve\Code\social_stream` at app startup; treat that repo as the primary runtime source.
- Do not treat `C:\Users\steve\Code\ssapp\resources\social_stream_fallback\main` as the source repo; it is a fallback mirror/bundle target.
- The `resources/social_stream_fallback/main` folder is replaced at build/update time and should be treated only as a backup, not the primary source.
- **Do not read, browse, edit, or add changes in `resources/social_stream_fallback` during normal app work.**  
  This folder is disposable/rebuilt on every build/update (`npm run update:fallback`), so spending time on it is not productive.

## Social Stream Payload Rules

- Donation-style chat rows should use `hasDonation` and optional `donoValue`; do not set `event: "donation"` just because a chat/tip row has a donation value.
- Use existing payload fields first. Only populate `meta` when there is additional structured data that downstream consumers actually need and no existing field handles it well.

## Build/Run Commands

| Command | Description |
|---------|-------------|
| `npm run start` | Start Electron app |
| `npm run start2` | Development mode (`--running-from-source`) |
| `npm run build` | Build for current OS |
| `npm run build:win32` | Build for Windows (NSIS + portable) |
| `npm run build:darwin` | Build for macOS (x64 + arm64) |
| `npm run build:linux` | Build for Linux (AppImage) |
| `npm run clean` | Remove dist folder |
| `npm run update:fallback` | Update Social Stream fallback bundle |

## Release Rules

- Read `RELEASE.md` before any release, deploy, tag, or artifact-upload work.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steveseguin/ssn_app](https://github.com/steveseguin/ssn_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
