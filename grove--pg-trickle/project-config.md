---
trigger: always_on
description: PostgreSQL 18 extension written in Rust using **pgrx 0.17.x** that implements
---

# AGENTS.md — Development Guidelines for pg_trickle

## Project Overview

PostgreSQL 18 extension written in Rust using **pgrx 0.17.x** that implements
streaming tables with incremental view maintenance (differential dataflow).
Targets PostgreSQL 18.x.

**Primary goals:** Maximum performance, low latency, and high throughput for
stream tables are the top priorities. Differential refresh mode must be used
wherever possible — full refresh is a fallback of last resort. The performance
and scalability of this extension should be world-class. Data loss is
unacceptable — correctness and durability of committed changes must never be
sacrificed for performance gains.

Key docs: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) ·
[docs/SQL_REFERENCE.md](docs/SQL_REFERENCE.md) ·
[docs/CONFIGURATION.md](docs/CONFIGURATION.md) ·
[INSTALL.md](INSTALL.md)

---

## Workflow — Always Do This

After **any** code change:

```bash
just fmt          # Format code
just lint         # clippy + fmt-check (must pass with zero warnings)
```

After changes to SQL-facing code, run the relevant test tier:

```bash
just test-unit         # Pure Rust unit tests (no DB)
just test-integration  # Testcontainers-based integration tests
just test-e2e          # Full extension E2E tests (builds Docker image)
just test-all          # All of the above + pgrx tests
```

> E2E tests require a Docker image. Run `just build-e2e-image` if the image is
> stale, or use `just test-e2e` which rebuilds automatically.

When you're done and have edited files always remember to output git commands
for staging and committing the changes. The git commit message should summarize
the changes made. Feel free to put discrete changes into separate git commit
commands. Do not commit directly to git unless the user explicitly says it is
fine.

Never create a new git branch unless the current branch is `main`.

When creating a pull request, always write the PR description to a temporary
file using the **`create_file` tool** (never a shell heredoc or `echo`), then
pass it to `gh` via `--body-file`.  Shell heredocs and terminal commands
silently corrupt Unicode characters and can pick up stale content from a
previous session's file at the same path.

**Guaranteed-safe workflow:**

1. Delete any stale file at the target path first:
   ```bash
   rm -f /tmp/pr_TICKETNAME.md
   ```

2. Use the `create_file` tool to write the description.  The file is written
   in UTF-8 and read directly by `gh --body-file`, so Unicode characters
   (math symbols, em-dashes, etc.) are safe to use.

3. Verify the file is clean before using it:
   ```bash
   python3 -c "
   with open('/tmp/pr_TICKETNAME.md') as f:
       body = f.read()
   print('lines:', body.count(chr(10)))
   print('ok:', '####' not in body)
   print(body[:120])
   "
   ```

4. Create or update the PR:
   ```bash
   gh pr create --title "..." --body-file /tmp/pr_TICKETNAME.md
   # or, to fix a garbled description:
   gh pr edit <number> --body-file /tmp/pr_TICKETNAME.md
   ```

5. Verify the live PR body is not garbled:
   ```bash
   gh pr view <number> --json body --jq '.body' | head -20
   ```

---

## Coding Conventions

### Error Handling

- Define errors in `src/error.rs` as `PgTrickleError` enum variants.
- Never `unwrap()` or `panic!()` in code reachable from SQL.
- Propagate via `Result<T, PgTrickleError>`; convert at the API boundary with
  `pgrx::error!()` or `ereport!()`.

### SPI

- All catalog access via `Spi::connect()`.
- Keep SPI blocks short — no long operations while holding a connection.
- **Always cast `name`-typed columns to `text`** when fetching into Rust
  `String` (e.g. `n.nspname::text`). The PostgreSQL `name` type (Oid 19)
  is not compatible with pgrx `.get::<String>()` (Oid 25).
- Catalog lookups must handle "not found" gracefully — CTEs, subquery aliases,
  and function-call ranges do not exist in `pg_class`. Return `Option` rather
  than `.first().get()` which panics on empty results.
- **Separate pure logic from SPI calls** so the decision logic can be
  unit-tested without a PostgreSQL backend (see `classify_relkind`,
  `strip_view_definition_suffix` for examples).

### Unsafe Code

- Minimize `unsafe` blocks. Wrap `pg_sys::*` in safe abstractions.
- Every `unsafe` block must have a `// SAFETY:` comment.

### Memory & Shared State

- Be explicit about PostgreSQL memory contexts.
- Use `PgLwLock` / `PgAtomic` for shared state; initialize via `pg_shmem_init!()`.

### Background Workers

- Register via `BackgroundWorkerBuilder`.
- Check `pg_trickle.enabled` GUC before doing work.
- Handle `SIGTERM` gracefully.

### Logging

- Use `pgrx::log!()`, `info!()`, `warning!()`, `error!()`.
- Never `println!()` or `eprintln!()`.

### SQL Functions

- Annotate with `#[pg_extern(schema = "pgtrickle")]`.
- Catalog tables live in schema `pgtrickle`, change buffers in `pgtrickle_changes`.

---

## Module Layout

```
src/
├── lib.rs          # Extension entry point, GUCs, shmem init
├── api.rs          # SQL-callable functions (create/alter/drop/refresh)
├── catalog.rs      # pgtrickle.pgt_stream_tables CRUD
├── cdc.rs          # Change-data-capture (trigger-based)
├── config.rs       # GUC definitions
├── dag.rs          # Dependency graph, topological sort, cycle detection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grove/pg-trickle](https://github.com/grove/pg-trickle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
