---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — ultrasearch

> Guidelines for AI coding agents working in this Rust codebase.

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

We only use cargo in this project, NEVER any other package manager. At the human overseer's instruction (2025-11-21), we now target the latest **nightly** Rust and track the latest versions of all crates (wildcard constraints). We ONLY use Cargo.toml for managing the project dependencies.

In general, you should try to follow all suggested best practices listed in the file `RUST_BEST_PRACTICES_GUIDE.md`

We load all configuration details from the existing .env file (even if you can't see this file, it DOES exist, and must NEVER be overwritten!). We NEVER use std::env::var() or other methods to get variables from our .env file other than using the dotenvy crate in this very specific pattern of usage (this is just an example but it always follows the same basic pattern):

```rust
use dotenvy::dotenv;
use std::env;

// Load .env file at startup (typically in main())
dotenv().ok();

// Configuration
let api_base_url = env::var("API_BASE_URL").unwrap_or_else(|_| "http://localhost:8007".to_string());
```

We use sqlx (async SQL toolkit) and diesel (ORM) for various database related functions. Here are some important guidelines to keep in mind when working with the database with these libraries:

Do:

- Create your connection pool with `sqlx::Pool::connect()` and use it across your application; the pool handles connection lifecycle automatically.
- Use `?` placeholder for parameters in queries to prevent SQL injection: `sqlx::query!("SELECT * FROM users WHERE id = ?", user_id)`.
- Use the query macros (`query!`, `query_as!`) for compile-time SQL verification when possible.
- Keep one database transaction per logical operation using `pool.begin().await?` and explicitly commit with `tx.commit().await?`.
- Use `fetch_one()`, `fetch_optional()`, or `fetch_all()` appropriately based on expected results.
- Explicitly handle migrations with sqlx-cli: `sqlx migrate run`.
- Use strong typing with `sqlx::types` for custom database types.
- On shutdown, connections are automatically closed when the Pool is dropped.

Don't:

- Don't share a single transaction across multiple concurrent tasks.
- Don't use string concatenation to build SQL queries (SQL injection risk).
- Don't forget to handle `Option<T>` for nullable columns properly.
- Don't mix sync and async database operations in the same codebase.
- Don't ignore error handling - database operations can fail for many reasons.
- Don't forget to enable the appropriate runtime and TLS features in Cargo.toml for sqlx.
- Don't use unwrap() on database results in production code - always handle errors properly.

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/ultrasearch](https://github.com/Dicklesworthstone/ultrasearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
