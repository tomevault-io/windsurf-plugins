---
trigger: always_on
description: > Guidelines for AI coding agents working in this hybrid Bash + Node.js codebase.
---

# AGENTS.md — giil

> Guidelines for AI coding agents working in this hybrid Bash + Node.js codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Bash + Node.js

This is a **hybrid project**: a Bash wrapper script with an embedded Node.js extractor.

### Bash Layer (giil, install.sh)

- Target **Bash 4.0+** compatibility. Use `#!/usr/bin/env bash` shebang.
- Use `set -euo pipefail` for strict error handling.
- Use **ShellCheck** to lint all scripts. Address all warnings at severity `warning` or higher.
- Ignore rules: `SC2155` (declare and assign separately), `SC2034` (unused variables in heredocs).

### Node.js Layer (extractor.mjs, embedded)

- The extractor is embedded in the giil script as a heredoc — regenerated fresh each run.
- Use ES modules (`"type": "module"` in package.json).
- Target **Node.js 18+**.

### Key Dependencies

| Dependency | Purpose |
|------------|---------|
| **Playwright** 1.40.0 | Browser automation (Chromium headless) |
| **Sharp** 0.33.0 | Image processing with MozJPEG compression |
| **exifr** 7.1.3 | EXIF metadata parsing and datetime extraction |
| **curl** | Installer and direct downloads (Dropbox) |
| **sips** (macOS) / **heif-convert** (Linux) | HEIC/HEIF conversion |
| **gum** (optional) | Beautiful terminal UI (banners, spinners, styled text) |
| **ImageMagick** (optional) | Image inspection (`identify` command) |

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself
- The embedded JavaScript heredoc is sensitive — maintain proper escaping

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.

**NEVER** create variations like:
- `giil_v2`
- `giil_improved`
- `install_enhanced.sh`

New files are reserved for **genuinely new functionality** that makes zero sense to include in any existing file. The bar for creating new files is **incredibly high**.

---

## Backwards Compatibility

We do not care about backwards compatibility—we're in early development with no users. We want to do things the **RIGHT** way with **NO TECH DEBT**.

- Never create "compatibility shims"
- Never create wrapper functions for deprecated APIs
- Just fix the code directly

---

## Compiler Checks (CRITICAL)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/giil](https://github.com/Dicklesworthstone/giil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
