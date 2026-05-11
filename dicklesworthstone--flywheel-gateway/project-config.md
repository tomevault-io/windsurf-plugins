---
trigger: always_on
description: > Guidelines for AI coding agents working in this TypeScript/Bun codebase.
---

# AGENTS.md — flywheel_gateway

> Guidelines for AI coding agents working in this TypeScript/Bun codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## RULE 2 - PUBLIC/PRIVATE SEPARATION

This is a **PUBLIC open source repository**. Never add business content here.

**All business content belongs in the sibling private repo** at `/data/projects/flywheel_private/`.

This follows the "Workspace Root, Sibling Repos" pattern:
```
/data/projects/                    (non-git workspace root)
├── flywheel_gateway/              (public repo - YOU ARE HERE)
└── flywheel_private/              (private repo - business content)
```

The public repo's `.gitignore` blocks `private_business/` as defense-in-depth, but the private repo should NEVER be nested inside this repo.

If you're unsure whether something is business content, ask first.

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

### DCG (Destructive Command Guard)

DCG is a high-performance Rust pre-execution hook that provides **mechanical enforcement** of command safety. Unlike these AGENTS.md instructions which you might ignore, DCG physically blocks dangerous commands before execution.

**What DCG blocks:**
- Git destructive ops: `git reset --hard`, `git push --force`, `git clean -f`
- Filesystem ops: `rm -rf` outside safe temp directories
- Database ops: `DROP`, `TRUNCATE`, `DELETE` without WHERE
- Container ops: `docker system prune`, `kubectl delete namespace`
- Cloud ops: destructive AWS/GCP/Azure commands

**If DCG blocks you:**
1. Do NOT attempt to bypass or rephrase the command
2. Read the block reason carefully
3. If you believe it's a false positive, ask the user for explicit approval
4. The user can allowlist specific patterns via the Gateway UI

DCG is your safety net—work with it, not against it.

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

## Toolchain: TypeScript & Bun

We only use **Bun** in this project, NEVER any other package manager or runtime.

- **Runtime:** Bun 1.3+ (package manager, bundler, test runner — now part of Anthropic)
- **Language:** TypeScript 5.9+ (strict mode enabled; 7.0 Go port coming)
- **Linting/Formatting:** Biome 2.0+ (linting, formatting, and type inference)
- **Module system:** ESM (`"type": "module"`)
- **Target:** ES2022

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `hono` (4.11+) | HTTP framework (ultrafast, Bun-native) |
| `drizzle-orm` (0.40+) | TypeScript-native ORM |
| `bun:sqlite` | Native SQLite (fast, zero-config) |
| `pino` | Structured logging |
| `@modelcontextprotocol/sdk` | MCP client/server integration |
| `@opentelemetry/*` | Distributed tracing (OTLP export) |
| `react` (19.2+) | UI framework (with React Compiler) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/flywheel_gateway](https://github.com/Dicklesworthstone/flywheel_gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
