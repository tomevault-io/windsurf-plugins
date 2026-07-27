---
trigger: always_on
description: Congrats, your user wants you to help them contribute to the Obsidian Claude Sidebar plugin. Your job is to help them produce a good contribution: a clean bug report, or a focused PR that fits what this plugin is, and is easy for the maintainer.
---

# Claude Sidebar - Agent Guide

Congrats, your user wants you to help them contribute to the Obsidian Claude Sidebar plugin. Your job is to help them produce a good contribution: a clean bug report, or a focused PR that fits what this plugin is, and is easy for the maintainer. 

Repo: https://github.com/derek-larson14/obsidian-claude-sidebar

## What this plugin does

Make agents feel native in Obsidian. The plugin runs the terminal that hosts the agent and routes Obsidian actions to it. It does not own anything else. The agent CLI (Claude Code, Codex, OpenCode, Gemini, Kimi Code, GitHub Copilot, Pi) handles auth, models, billing, and its own UI.

## Two things you can help the user do

1. **File a bug report.** You hit something but don't have a fix. Diagnose (Step 1), then open an issue with the diagnostic block.
2. **Open a PR.** (preferred) You have a fix or a small feature you want to propose. Diagnose, fix, test, then PR.

## Step 1: Diagnose

Run through this list. The maintainer will ask for everything in it on every issue, so paste the block into your report. If you're opening a PR, use the same info as context for figuring out the fix.

**Required (issues):**
- Plugin version (Settings → Community plugins → Claude Sidebar - or read `manifest.json`)
- Install method (BRAT / manual / Obsidian community store)
- CLI backend and version (`claude --version` or equivalent for your backend)
- Obsidian version + installer version (Settings → About - both numbers)
- OS and how Obsidian was installed
  - Linux: snap, AppImage, deb, Flatpak
  - Windows: installer or Microsoft Store
  - Mac: dmg or Homebrew
- Other plugins active that touch hotkeys, layout, or terminals (BRAT, iconize, Vimrc). Load-order interactions cause real bugs.

**Required for crashes or freezes:**
- Developer console output. Open with `Cmd-Option-I` on Mac, `Ctrl-Shift-I` on Windows/Linux. Copy any errors from the Console tab.

**Already tried (check what applies):**
- Restarted Obsidian
- Disabled other plugins
- Updated to latest plugin version
- Tested in a fresh empty vault

**Windows-specific:**
- Read the actual Python error from the terminal output first - `terminal_win.py` prints `sys.executable` and a pip command on import failure
- If no obvious error: output of `where python` and `where py`, and whether you're using pyenv, conda, or a virtualenv

If your agent has shell access, it can collect most of this for you.

## Step 2a: File a bug report

If you don't have a fix, open an issue at https://github.com/derek-larson14/obsidian-claude-sidebar/issues with the diagnostic block above plus:
- What happened
- What you expected
- Steps to reproduce (numbered, exact)

## Step 2b: Open a PR

The fast loop: edit `main.js` in your vault, reload Obsidian (`Cmd-R` on Mac, `Ctrl-R` on Windows/Linux), confirm the fix, copy to a fork, open a PR against `main` (the agent can help with much of this).

If you installed via BRAT, disable auto-update for this plugin while you iterate - BRAT will overwrite your edits.

If you're going to iterate often, symlink your fork into a vault to skip the copy step:

```bash
ln -s /path/to/your/fork /path/to/vault/.obsidian/plugins/claude-sidebar
```

Test in both the sidebar pane and a main-area tab - different layout code paths.

### Python fixes (PTY backend)

If your bug looks like one of these, it's likely Python:

- Orphaned processes after closing a tab, idle CPU spin, kill signals not propagating
- CJK or multi-byte character corruption in terminal output
- Terminal won't resize at all (vs. resizing but rendering wrong)
- Windows: pywinpty errors, ConPTY-specific issues
- Terminal capability queries (DA1/DA2/DSR) breaking cmd.exe or the shell

The two files:

- `terminal_pty.py` - Mac and Linux. Forks a child shell via `pty.fork()` (the shell is whatever the user has configured - not always bash), relays stdin/stdout, parses RESIZE escape sequences, sends SIGTERM-then-SIGKILL to the process group on cleanup.
- `terminal_win.py` - Windows. Spawns via pywinpty, handles UTF-8 multi-byte sequences across read boundaries, filters terminal protocol responses (DA1/DA2/DSR) so cmd.exe doesn't break.

These files aren't installed in your vault. They're embedded as base64 in `main.js`. To fix Python you need a clone:

1. Fork and clone the repo.
2. Edit the relevant file.
3. Run `bash build.sh` to re-embed as base64 into `main.js`.
4. Copy the rebuilt `main.js` into your vault, reload Obsidian, and test.
5. Commit, push, open a PR.

### Rules for any PR

- **Make ONE change at a time.**
- **Don't bump `manifest.json`.**
- **Test the surface you touched, plus one nearby.**
- **In the PR description, include:**
  - What was wrong
  - What you changed
  - What you tested it on (which OS, which CLI backend)

## Things we like to see:

1. **Routing an Obsidian-native action into the terminal** - file paths, selections, clipboard images, folder context menus, ribbon actions.
2. **Bug fixes / regressions** that break the existing experience.

## Things we do not like to see:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derek-larson14/obsidian-claude-sidebar](https://github.com/derek-larson14/obsidian-claude-sidebar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
