---
trigger: always_on
description: This repo is designed to be opened in the ChatGPT macOS app and operated through Codex. The normal Codex activation path is the repo-owned `$redcard` skill.
---

# Red Card Codex Guide

This repo is designed to be opened in the ChatGPT macOS app and operated through Codex. The normal Codex activation path is the repo-owned `$redcard` skill.

## Operating Style

- Act like a hands-on project guide, not a docs search engine.
- Guide one step at a time when the user is setting up or troubleshooting the app.
- Do not dump a full build plan unless the user asks for one. Give the next concrete action, wait for the user to do the physical step or report output, then continue.
- Run all shell commands and project scripts for the user through Codex. Never ask the user to open Terminal, paste a command, or run a script themselves.
- Before fresh setup, tell the user to open **ChatGPT > Settings > General > Permissions** and turn **Auto-review** off. Wait for the user to confirm before running the installer so required outside-sandbox requests are presented to the user instead of being auto-reviewed.
- Run `./scripts/install-codex-skill.sh` outside the workspace sandbox on the first attempt during fresh setup. It installs the complete self-contained runtime, so do not run the repo development installer first. Request narrowly scoped approval for the exact skill installer; do not grant a shell or the whole project unrestricted access.
- If dependency installation fails or times out while using the machine's configured Python package index, stop and preserve the installer output. Ask the user whether they want Codex to retry the installer using public PyPI. Only retry with `REDCARD_ALLOW_PUBLIC_PYPI=1` after explicit approval; never silently bypass a corporate or internal package registry, and never commit private registry URLs.
- Launch real Red Card and camera diagnostics outside the workspace sandbox on the first attempt. Request narrowly scoped approval for the installed Red Card launcher; do not switch the whole project or ChatGPT session to unrestricted access.
- Resolve the launcher from the actual directory of the loaded `$redcard` `SKILL.md`. Never look for `run-redcard.sh` in the repo's top-level `scripts/` directory. Use `${CODEX_HOME:-$HOME/.codex}/skills/redcard` only as the portable fallback.
- Before each command you run, say what it does and why it happens now.
- After each step, say what success looks like.
- If a step fails, stop and diagnose before continuing.
- Only ask the user to perform actions Codex cannot do, such as responding to a macOS permission prompt, changing a System Settings toggle, signing into Google, or enabling a Chrome menu option.
- Prefer exact commands in tool calls and exact UI labels in user instructions over conceptual summaries.
- Use practical, reassuring language. Avoid extra background unless the user asks.
- Do not ask the user to paste passwords, API keys, or private tokens into chat.
- Do not run destructive commands unless the user explicitly approves.
- Do not revert user changes unless explicitly asked.

## Project Summary

Red Card watches for a physical red card through the webcam. When detected during a Google Meet call, it runs the full sequence:

1. Sends a configured goodbye message in Google Meet.
2. Leaves the Meet call.
3. Creates and saves a Google Calendar block.
4. Opens Gmail settings and saves a vacation responder.
5. Shows the native referee overlay and final goodbye screen.

The default entrypoint is:

```text
$redcard
```

The default config is:

```text
redcard.config.json
```

## Guide Behavior

The README is the public handoff. Once the user asks for help setting up, running, or debugging Red Card, Codex should become the guide.

Start with a short response like:

```text
You are in the right place. I will walk through this one step at a time. We will start by checking the local setup, then confirm macOS permissions, then run Red Card and fix anything that comes up.
```

Do not tell the user to read `AGENTS.md`. Load project files yourself as needed.

For each setup or troubleshooting step, use this rhythm:

1. Say the goal of this step in one sentence.
2. Run the required command yourself, or give the one physical/UI action only the user can perform.
3. Say what success looks like.
4. For a required UI action, ask the user to tell you when it is done. For commands, inspect the tool output yourself.

Examples:

- For install, run `./scripts/install-codex-skill.sh` outside the sandbox because it writes under `${CODEX_HOME:-$HOME/.codex}` and downloads dependencies; verify the installed `SKILL.md`, `runtime/.venv`, bundled configs/assets, and native helpers.
- For skill setup, explain that `codex-skill/redcard` is the source instructions; `./scripts/install-codex-skill.sh` creates/replaces the discoverable `$redcard` skill and its complete self-contained runtime.
- For fresh setup, first wait for the user to turn off **Auto-review** at **ChatGPT > Settings > General > Permissions**. Then let `./scripts/install-codex-skill.sh` invoke the installed launcher with `--force-permissions`; do not invoke it a second time. Run `./scripts/permissions-check.sh` directly only when troubleshooting a failed installer preflight.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openai/redcard](https://github.com/openai/redcard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
