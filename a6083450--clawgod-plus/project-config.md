---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Common Commands

### Bun tests and compatibility checks

Bun 1.3.14 or newer is the only installed JavaScript runtime required by this repository. Run focused checks with Bun:

```bash
bun tests/bun-only-policy.mjs
bun tests/installer-bun-runtime.mjs
bun tests/installer-ripgrep.mjs
bun tests/installer-plugin-dependencies.mjs
```

Run the complete local regression set without installing ClawGod Plus:

```bash
for test_file in tests/*.mjs; do
  bun "$test_file" || exit 1
done

bash -n dist/unix/install.sh
git diff --check
```

The compatibility workflow in `.github/workflows/compat-daily.yml` runs the Unix installer end-to-end and smoke-tests the generated command. Do not use `bash dist/unix/install.sh` as a casual local test: it writes to `~/.clawgod`, backs up and replaces the `claude` command, and creates `clawgod` launchers.

The Bun-only CI compatibility workflow runs Linux x64 and macOS 26 ARM64 on schedule and relevant changes; Windows is change-driven only. GitHub Actions run native Node.js 24 actions, which are not a product runtime dependency.

The Windows lifecycle assertions run in GitHub Actions with PowerShell JSON APIs and Bun. Do not describe them as locally native-verified when `pwsh` is unavailable on the current machine.

### Generated installer build

`dist/unix/install.sh` and `dist/win/install.ps1` are deterministic generated release artifacts. `src/` is the canonical source of truth: shared JavaScript lives under `src/generic/`, platform lifecycle and launcher code under `src/unix/` and `src/windows/`, and thin templates under `src/template/`. The generated installers must never be edited by hand — regenerate and verify them with:

```bash
bun build.mjs
bun build.mjs --check
```

`bun build.mjs --check` fails when the checked-in installers are stale relative to `src/`.

### Installer usage

README-documented user install commands:

```bash
curl -fsSL https://github.com/A6083450/clawgod-plus/releases/latest/download/install.sh | bash
```

```powershell
irm https://github.com/A6083450/clawgod-plus/releases/latest/download/install.ps1 | iex
```

Shell and PowerShell are operating-system command entry points, not JavaScript runtimes. The installer runs with Bun and privately installs and verifies ripgrep 15.2.0; users do not need a system ripgrep.

Useful local installer options:

```bash
bash dist/unix/install.sh --version <version>
bash dist/unix/install.sh --uninstall
```

Windows uninstall:

```powershell
.\dist\win\install.ps1 -Uninstall
```

`claude update` routes through the ClawGod Plus installer, fetches the requested Anthropic Claude Code package from the npm Registry, re-extracts and re-patches it, then rewrites the launchers.

### Enhancement selection

ClawGod Plus resolves a persisted, optionally interactive choice of 21 enhancements. The stable IDs, in manifest order, are `chrome`, `computer-use`, `design-canvas`, `agents`, `planning`, `voice`, `auto-mode`, `unrestricted-tools`, `paste-images`, `privacy`, `branding`, `classifier-fail-open`, `cleanup-period`, `disable-collapse-read-search`, `enable-keybindings`, `file-read-limit`, `transcript-dialog-replay`, `unlock-ultracode` (patches), then `claude-hud`, `claude-mem`, `superpowers` (plugins). Selection is persisted as strict JSON at `~/.clawgod/enhancements.json` with the schema `{ "schemaVersion": 1, "mode": "all" | "custom", "enabled": [...] }`.

Direct local installers accept `--enhancements <csv>` / `--choose-enhancements` (Unix) and `-Enhancements <csv>` / `-ChooseEnhancements` (PowerShell). Running the installer directly in a terminal auto-prompts a quick choice (all / core-only / custom menu) via stdin-TTY detection; the menus are key-driven (`↑`/`↓` move, `Space` toggles, `Enter` confirms, `Esc` returns to the parent menu and cancels the install at the top level); piped installs, CI, and `claude update` never prompt (the update patch marks its installer spawn with `CLAWGOD_NONINTERACTIVE=1`) and reuse the saved selection, defaulting to all enhancements. Disabling `claude-hud` or `claude-mem` restores the configuration ClawGod owns, while disabling `superpowers` never deletes the user's installed plugin.

The user restored upstream Cometix ASR under the existing `voice` enhancement. Preserve the original adapter; only adapt ClawGod vendor paths and newer command/transport availability checks. The installer fetches commit-pinned, SHA-256-checked native files on supported platforms and validates Bun loading only. Never record audio or call the native startSession/ensureDid as an installation smoke test. This is network-backed ASR, not offline transcription. `CLAUDE_CODE_ASR=0` or the `voice-asr-backend` runtime switch disables this transport; do not modify unrelated account/compliance checks.

## Project Architecture

ClawGod Plus is an installer-driven runtime patch project for official Claude Code, not a conventional application library. The repository has two parts:

1. **Self-contained installers and runtime patcher**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [A6083450/clawgod-plus](https://github.com/A6083450/clawgod-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
