---
trigger: always_on
description: Instructions for agentic coding assistants working on the Migrondi codebase.
---

# AGENTS.md

Instructions for agentic coding assistants working on the Migrondi codebase.

## Project Overview

Migrondi is a SQL migrations tool: you write versioned `.sql` files and it tracks which have been applied to a database — applying pending ones ("up") or rolling applied ones back ("down"), plus dry-run, list, and per-file status checks. It runs each migration inside a transaction (with an opt-out `manualTransaction` flag for statements like `CREATE INDEX CONCURRENTLY`) and supports **SQLite, SQL Server (MSSQL), PostgreSQL, and MySQL** over ADO.NET (see the `MigrondiDriver` DU).

The repo ships three independently useful components built on one engine:

- **`Migrondi.Core`** — the packable migration library. The `IMigrondi` service (built via `Migrondi.MigrondiFactory`) coordinates the file system and database: create migrations, apply/rollback (optionally a count), dry-run, and list status. Migration content is read through a pluggable `IMiMigrationSource` — the default reads SQL from disk, but you can supply a custom source (HTTP, S3, blob storage, …). The public API is CLS-friendly (sync + async overloads, `Result` extensions), so it embeds cleanly into F#/C#/VB applications.
- **`Migrondi`** — the CLI, distributed as a `dotnet` global tool (`migrondi`) and as self-contained single-file binaries for Windows/macOS/Linux (x64 + arm64). Commands: `init`, `new`, `up`/`apply`, `down`/`rollback`, `list`/`show`, `status` (each with aliases).
- **`MigrondiUI`** — an Avalonia desktop app that also runs as an **MCP (Model Context Protocol) server**. As a UI it manages multiple migration "projects" (on-disk local projects and virtual projects stored in its own database). In MCP mode it exposes those projects and migration operations (create/update/delete migrations, dry-run, apply, rollback, import/export) as tools, so an AI agent can drive migrations over MCP (MCP operations target virtual projects).

The codebase uses F# with `.fsi` signature files, MSTest for tests, and Fantomas for formatting.

## Imperatives

1. **NEVER PUSH WITHOUT PERMISSION.** Always ask before pushing to the remote.
2. **NEVER FORCE PUSH.** Tell the user they have to force push instead of you.
3. **Always run `dotnet fantomas .` before committing code.** Format all F# files before staging.
4. **Never use `Option.get` or `ValueOption.get`.** Always pattern match (`match`, `function`, `if ... then`) or use safe alternatives (`Option.defaultValue`, `Option.map`, `Array.choose`, etc.) to handle option values. Unchecked `.get` calls crash at runtime on `None`.
5. Pull requests made with the `gh` command should use a markdown file as the PR body, not inline escaped markdown strings.

## Project Structure

- `src/Migrondi.Core` — the packable migration library/engine.
- `src/Migrondi` — the CLI (global tool + self-contained binaries).
- `src/MigrondiUI` — the Avalonia app and MCP server.
- `src/Migrondi.Tests`, `src/MigrondiUI.Tests` — test projects (not packable).
- `samples/` — sample usage.
- `docs/` — the [FsDocs](https://fsprojects.github.io/FSharp.Formatting/) documentation site.

> Target frameworks, package versions, and similar build details live in each project's `.fsproj` and `Directory.Build.props` — read those directly rather than relying on this file, since they change over time.

## Migration File Format

Migration `.sql` files are **parsed by strict regex**, not free-form text. The `-- MIGRONDI:` lines and the `UP`/`DOWN` delimiters carry data — never delete, reorder, reformat, or "tidy" them (the files themselves say `-- Do not remove MIGRONDI comments.`). The authoritative parser is `src/Migrondi.Core/Serialization.fs` (`Migration.EncodeText` / `Migration.DecodeText`); filename matching lives in `Library.fsi` (`Matcher.V0` / `Matcher.V1`). When in doubt, read those.

### V1 format — current; used for all new migrations

```sql
-- MIGRONDI:NAME=add_users
-- MIGRONDI:TIMESTAMP=1586550686936
-- MIGRONDI:ManualTransaction=true
-- ---------- MIGRONDI:UP ----------
CREATE TABLE users (id INTEGER PRIMARY KEY);
-- ---------- MIGRONDI:DOWN ----------
DROP TABLE users;
```

- Metadata lines are `-- MIGRONDI:KEY=VALUE`; both `KEY` and `VALUE` may only contain `[a-zA-Z0-9_-]+`.
  - `NAME` and `TIMESTAMP` (an `int64`) are required.
  - `ManualTransaction=true|false` is optional — emitted only when `true`; absent or any non-`true` value means `false`.
- Section delimiters must be **exactly** `-- ---------- MIGRONDI:UP ----------` and `-- ---------- MIGRONDI:DOWN ----------` (mind the dashes). "Up" content is everything between the two delimiters; "Down" (rollback) content is everything after the DOWN delimiter. Both are trimmed.
- Set `ManualTransaction=true` only for statements that can't run inside a transaction (e.g. `CREATE INDEX CONCURRENTLY`); otherwise omit it so the migration runs in a transaction and rolls back on failure.

### V0 format — pre-v1, deprecated (read-only)

```sql
-- ---------- MIGRONDI:UP:1586550686936 ----------
CREATE TABLE users (id INTEGER PRIMARY KEY);
-- ---------- MIGRONDI:DOWN:1586550686936 ----------
DROP TABLE users;
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AngelMunoz/Migrondi](https://github.com/AngelMunoz/Migrondi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
