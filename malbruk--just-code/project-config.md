---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A community VS Code extension that reimplements Anthropic's official **Claude Code**
extension on top of the **Claude Agent SDK** (`@anthropic-ai/claude-agent-sdk`).
It provides an agentic chat panel (activity-bar view + editor-tab mode) with the
full Claude Code toolset, streaming, tool cards, inline diffs, permission prompts,
editor context, @-file mentions, slash commands, model picker, usage, history, and
subscription/API-key auth.

Not affiliated with Anthropic. TypeScript throughout, `strict: true`.

## Shell Environment

This machine runs Windows/PowerShell. Never use bash-only syntax (here-strings, single-quote
escaping, `$'...'`) in Bash tool calls. For multi-line git commit messages, write the message
to a temp file and use `git commit -F <file>`.

## Commands

```bash
npm install
npm run compile      # build BOTH bundles via esbuild (dist/extension.js + media/webview.js)
npm run watch        # esbuild watch
npm run check-types  # tsc --noEmit -p tsconfig.json   (run this after edits)
npm run vsix         # package .vsix  (npx @vscode/vsce package)
```

Debug: press **F5** ("Run Extension" launch config → runs with `--disable-extensions`).
There is no unit-test runner; verify with the scripts below.

### Verifying changes (no VS Code GUI needed)

- `npx tsc --noEmit -p tsconfig.json` — must be clean (whole project, both halves).
- `node esbuild.js` — must build both bundles with no errors.
- `node scratch/activate-test.js` — stubs the `vscode` module, requires the built
  `dist/extension.js`, calls `activate()`, and asserts every command declared in
  `package.json` is registered and the `just-code.chat` view provider registers.
  Run this after touching activation, commands, or the manifest.
- `node scratch/usage-logic-test.mjs` — bundles `src/agent/usage.ts`, issues a live
  `/usage` control request, and pins the window mapping + banner thresholds.
- `node scratch/account-dialog-test.mjs` — renders `webview-ui/src/account.ts` against
  a DOM shim: markup, escaping, the Day/Week toggle, the API-key and error states.
  Run both after touching account/usage code.
- `node scratch/delete-session-test.mjs` — creates a synthetic transcript under a temp
  workspace's project dir, then pins the real SDK's `deleteSession`: the `.jsonl` and the
  subagent subdirectory are unlinked, `listSessions` stops returning it, and a second
  delete throws. Run after touching history deletion.
- `node scratch/mention-chip-test.mjs` — drives the real `Composer` against a DOM shim:
  `@`-mention chip painting (offsets, escaping, non-mentions) and the completion
  popup's folder drill-down. `node scratch/file-completions-test.mjs` pins the host
  side of the same feature (folder derivation, trailing slashes, name/detail split).
  Run both after touching `composer.ts` or `context/completions.ts`.
- `node scratch/slash-usage-test.js` — drives the real `SessionManager.handleMessage`
  with a stubbed `vscode` to check `/usage` routing. It pins the config stub to an
  API-key session with no key, so an unknown command can't fall through and bill a
  real turn. Extend it when adding a client-side slash command.
- Native binary / auth path: `node -e` resolving `@anthropic-ai/claude-agent-sdk-<plat>/claude(.exe)`
  and running `claude auth status --json` (see git history / scratch for the exact snippet).

### Verifying UI changes

VS Code loads the *installed* extension copy, not the working tree. After any webview/CSS/icon
change, run the full build and reinstall the VSIX (or reload the Extension Development Host)
before claiming the change is visible. Never state "it should now show X" without a
build + reinstall step.

## Debugging

Before proposing a cause for a numeric/behavioral bug, gather real evidence (debug logs, actual
values, screenshots). Do not assert a cause from assumption — say "I need to see X" instead.

## Architecture

Two isolated halves that communicate ONLY through a typed message protocol:

```
Extension host (Node, CJS)  ⇄  src/shared/protocol.ts  ⇄  Webview UI (browser, IIFE)
        src/**                    (frozen contract)            webview-ui/**
```

- **`src/shared/protocol.ts` is the frozen contract.** Imported by BOTH sides. It
  defines `HostToWebview` / `WebviewToHost` messages and all view-model types
  (`ChatMessage`, `ToolUseView`, `DiffView`, `PermissionRequest`, `Attachment`,
  `AuthInfo`, `WebviewState`, …). It must stay dependency-free (no `vscode`, no DOM).
  Change it deliberately and update both sides + `check-types`.
- **Host → webview** posts `HostToWebview`; **webview → host** posts `WebviewToHost`.
  Never invent a message not in the protocol — add it to the union first.
- **esbuild builds two bundles** (`esbuild.js`): the host (`src/extension.ts` →
  `dist/extension.js`, CJS, `platform:node`) and the webview
  (`webview-ui/src/main.ts` → `media/webview.js`, IIFE, `platform:browser`).
- The webview is **vanilla TypeScript + DOM, no frameworks, no external/CDN
  resources** (strict CSP). All styles live in `media/webview.css` and use VS Code
  theme CSS variables only.

### Agent SDK integration (the important gotchas)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [malbruk/just-code](https://github.com/malbruk/just-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
