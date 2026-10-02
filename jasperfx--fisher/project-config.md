---
trigger: always_on
description: handles those whether the aggregate declares an `Apply` or not: correct about what the projection
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Fisher?

SQLite-backed Event Store and lightweight Document Database in the Critter Stack. A deliberate
subset of Marten (PostgreSQL) and Polecat (SQL Server), built on Weasel.Sqlite for schema management
and Weasel.Storage for the shared closed-shape document/event runtime.

**Fisher is early and incomplete.** See "Current state" below before assuming a feature exists,
[ROADMAP.md](ROADMAP.md) for what comes next and in what order, and [HANDOFF.md](HANDOFF.md) for the
compliance scoreboard and the current state of play.

## Commands

```bash
dotnet build fisher.slnx
dotnet test fisher.slnx                    # both TFMs; they run serially on purpose
dotnet test fisher.slnx -f net10.0         # one TFM

# One test class or method (Microsoft Testing Platform — note the bare `--`)
dotnet test src/Fisher.Tests/Fisher.Tests.csproj -f net10.0 -- --filter-method "*appending_events*"

# The documentation site
npm install
npm run docs                               # mdsnippets + dev server on :5173
npm run docs-build                         # a dead internal link fails the build
```

No database server is required — tests create throwaway SQLite files via
`TemporaryDatabase.Create()`.

Documentation code samples are compiled — see "The documentation site" below before editing a page's
code block.

## Architecture

### The three-layer split

Fisher owns very little storage code. The division is worth internalizing before adding anything:

- **JasperFx / JasperFx.Events** — event/projection/daemon abstractions. Implement its interfaces;
  do not reinvent them. `EventGraph` derives from `JasperFx.Events.EventRegistry`.
- **Weasel.Sqlite** — schema management, table definitions, migrations, the PRAGMA-applying
  `SqliteDataSource`, and `CommandBuilder`. All DDL goes through it; no hand-written CREATE TABLE.
  **All raw data access goes through it too** — see below.
- **Weasel.Storage** — the dialect-neutral closed-shape document + event storage runtime extracted
  from Marten. Fisher supplies two dialects (`SqliteStorageDialect<TId>`,
  `SqliteEventStoreDialect`) and the runtime does the rest. **Prefer extending the dialects over
  writing bespoke storage code.**

### Raw data access goes through Weasel.Sqlite first

**Reach for Weasel.Sqlite before writing ADO.NET by hand, and deviate only where it genuinely cannot
express what is needed.** Connections come from `SqliteDataSource` (via
`FisherDatabase.OpenConnectionAsync`), never from `new SqliteConnection(...)` in production code — the
data source is what applies the store's PRAGMA settings and what holds an in-memory database alive.
Statements are built with `Weasel.Sqlite.CommandBuilder` rather than by concatenating SQL and adding
parameters by hand, so parameter binding and placeholder handling stay in one implementation. Schema
work goes through Weasel's table definitions and migrations, which is what makes
`AutoCreate.None` honoured everywhere for free rather than at each call site's discretion.

**When Weasel.Sqlite cannot do it, the workaround is local, commented, filed upstream, and removed
when the fix ships.** That cycle has now run twice and closed twice, and **both removals are the
point of the rule** rather than a tidy-up:

- `FisherCommandBuilder` existed because `CommandBuilder` did not declare
  `Weasel.Core.ICommandBuilder`. Gone since weasel#424 shipped in 9.23.2 — do not reintroduce it.
- `DocumentTable.ConfigureQueryCommand` overrode Weasel's `pragma_table_info` query because it omits
  generated columns. Gone since [weasel#426](https://github.com/JasperFx/weasel/issues/426) shipped
  in **9.24.0** — do not reintroduce that either.

**The second one shows what the rule is actually protecting against**, because it was not removed on
the bump that fixed it and the cost arrived one release later. Weasel 9.25.0 added a fifth statement
(triggers) to the metadata query; Fisher's override still emitted four, and the override and the
reader that consumes it are one contract. The result was not a stale-but-harmless copy — it was
`ArgumentOutOfRangeException` out of `readForeignKeysAsync`, a result-set misalignment with nothing in
the message about Fisher or about generated columns. A deviation with no issue behind it is a
deviation nobody will ever remove; a deviation whose issue has *closed* is worse, because it now
diverges silently from the thing it was copied from.

**Reusable data-access helpers belong in Weasel.Sqlite eventually, so write them where they can move.**
The test is whether the helper would be equally correct for any SQLite consumer or whether it encodes
one of *Fisher's* storage decisions. `SqliteTimestamp`, `SqliteParameterValue`'s conversions, the row
readers and `FisherTableNaming` are all the latter — they are about the shape of Fisher's data, not
about SQLite, and they stay. Anything that is really "how you do X against SQLite" should be built as
a self-contained piece with no Fisher types in its signature, so pushing it upstream later is a file
move rather than a rewrite.

### SQLite divergences from Marten/Polecat


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JasperFx/fisher](https://github.com/JasperFx/fisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
