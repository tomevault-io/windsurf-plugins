---
trigger: always_on
description: > Guidelines for AI coding agents working in this Bun + TypeScript codebase.
---

# AGENTS.md — source_to_prompt_tui

> Guidelines for AI coding agents working in this Bun + TypeScript codebase.

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

## Toolchain: Bun & TypeScript

We only use **Bun** in this project, NEVER any other package manager.

- **Runtime:** Bun 1.3+ (latest)
- **Language:** TypeScript with strict mode
- **Target:** ES2020
- **Module system:** ESNext with bundler resolution
- **JSX:** react-jsx
- **Lockfile:** `bun.lock` only — never introduce `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`
- **Forbidden package managers:** `npm`, `yarn`, `pnpm`

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `ink` | React renderer for terminal interfaces |
| `ink-gradient` | Gradient text rendering |
| `ink-spinner` | Terminal spinner component |
| `ink-syntax-highlight` | Syntax-highlighted code in terminal |
| `ink-text-input` | Text input component |
| `react` | UI component model (v18) |
| `tiktoken` | Token counting (cl100k_base encoding) |
| `terser` | JavaScript minification |
| `csso` | CSS minification |
| `html-minifier-terser` | HTML minification |
| `ignore` | .gitignore-style pattern matching |

### Build Commands

```bash
bun run dev        # Run the TUI from source
bun run typecheck  # TypeScript typecheck (no emit)
bun run build      # tsc -> dist/
bun run build:bin  # bun build --compile -> dist/s2p
bun run build:all  # Cross-platform builds (macOS, Linux, Windows)
```

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.

**NEVER** create variations like:
- `indexV2.tsx`
- `index_improved.tsx`
- `index_enhanced.tsx`

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
> Source: [Dicklesworthstone/source_to_prompt_tui](https://github.com/Dicklesworthstone/source_to_prompt_tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
