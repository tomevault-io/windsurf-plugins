---
trigger: always_on
description: handles both wheel axes natively (`UpdateScroll`). Don't reintroduce a
---

# pgNimbus — project memory

## What this is

A fast, open-source PostgreSQL GUI client (.NET 10 + Avalonia 12), MIT
licensed. Windows is the primary target; the core engine stays
cross-platform-capable. The thesis: **truly fast + open source + modern UI** —
a gap none of pgAdmin/DBeaver (heavy), TablePlus (fast but paid/closed), or
HeidiSQL (fast but dated, MySQL-first) fill. pgNimbus aims for HeidiSQL's
speed with TablePlus's polish, PostgreSQL-first.

## Keep this file current

Whenever a change touches something this file documents — tech stack
versions, architectural rules, coding conventions, the sandbox bootstrap
steps — update the corresponding section in the same commit/PR. Treat a
stale `CLAUDE.md` (e.g. it still saying "Avalonia 11" after an upgrade to
12) as a bug, not a nitpick: it's the first thing a fresh session reads,
and wrong project memory is worse than none.

## The sibling project, and what is shared with it

kubeNimbus (`X:\source\kubeNimbus`, normally checked out beside this repo) is the
same product for Kubernetes, and the two must look and behave like one family.
The shared half lives in **[`shared/nimbusUi`](shared/nimbusUi/)** — a git subtree
of [nimbusUi](https://github.com/Shman4ik/nimbusUi), referenced as an ordinary
`ProjectReference`:

- `Theme/Tokens.axaml` — the palette, radii, scrollbars, Fluent resource overrides.
- `Theme/Icons.axaml` — the MDI glyphs both apps draw.
- `Theme/Theme.axaml` — the shared style classes (`card`, `layer`, `chip`,
  `toolbar`, `searchpill`, `statusBar`, …).
- `Theme/Controls.axaml` — the Fluent **control** retheming: `TextBox`/`ComboBox`/
  `NumericUpDown` radius and brand text selection, `SelectableTextBlock`,
  `ListBox`/`ListBoxItem`/`TreeView`/`TreeViewItem` rounded rows, `DataGrid` soft
  rules, the `.soft` and `.soft.danger` button families, `ToggleButton.soft`, the
  chip checked/active washes, `TabControl`. **These moved out of
  `Styles/Theme.axaml`**, where they had been defined for pgNimbus alone: kubeNimbus
  had none of them and was drawing stock Fluent inputs, lists and grids next to
  these, which is what made the two apps stop looking like one family. Change them
  there, not here. What is left in this app's own `Styles/Theme.axaml` is `TabItem`,
  `TabControl.segmented` and the AvaloniaEdit completion/search themes — all
  genuinely pgNimbus's (see DESIGN.md's not-shared table).
- `Chrome/` — the one-bar window chrome and its drawn caption buttons.
- `Hotkeys.cs` — Ctrl/Cmd resolution; `PgNimbus.App.Hotkeys` forwards to it.
- **[`DESIGN.md`](shared/nimbusUi/DESIGN.md) — the UI rules, single source.**

Three rules about it:

1. **A change to a shared surface is a change to both apps.** Edit the files in
   place, build pgNimbus, then `git subtree push --prefix shared/nimbusUi`, pull
   it into kubeNimbus and build that too. Both working copies are normally open
   side by side, so this is one session's work, not a follow-up ticket. The PR
   template asks for the paired PR.
2. **The membership test is "can it be described without naming Postgres?"** If
   yes it probably belongs up there; if no it stays here. When in doubt leave it
   here — a wrong thing pulled up has to be un-shared against two consumers.
3. **`DESIGN.md` owns the rule text; this file owns the evidence.** Don't restate
   a shared rule here in full — that is exactly how the two files started
   disagreeing (the same red carried two names and two values for months).

## Hard architectural rules

1. **`PgNimbus.Core` has zero Avalonia/UI dependencies.** Its only packages are
   `Npgsql`, `System.Security.Cryptography.ProtectedData` (the DPAPI credential
   store) and `SSH.NET` (the tunnel) — all headless. Anything UI-related belongs
   in `PgNimbus.App`. This keeps the engine reusable for a future CLI or test
   harness — don't leak `Avalonia.*` or `CommunityToolkit.Mvvm` types into
   `Core`.
2. **Streaming + cancellation are non-negotiable.** `QueryEngine.ExecuteAsync`
   returns result rows via `IAsyncEnumerable<RowBatch>` in ~200-row batches so
   the UI can render before the full result set arrives. Every execution
   takes a `CancellationToken` and must actually stop mid-flight, not just at
   the start. The one deliberate exception: inside an explicit transaction
   (`BeginTransactionAsync`), statements run on the single held session
   connection and return a fully-materialized `MaterializedResultSet` instead —
   a lazily-streaming reader would pin that connection open and block the next
   statement in the transaction. A failed statement inside a transaction
   auto-rolls-back the block (so the connection never lingers in Postgres's
   aborted-transaction state), and `TransactionStateChanged` is how the App's
   "in transaction" indicator stays in sync no matter which path changed it.
   Auto-reconnect (2026-07): `QueryEngine` classifies a failure as connection
   loss (Postgres class-08 `SqlState`s / an admin or crash shutdown, or an
   `NpgsqlException` wrapping a socket/IO exception — deliberately not
   `TimeoutException`, which Npgsql also uses for command timeouts and pool
   exhaustion where a silent re-run could double-apply work) versus an
   ordinary statement error, and on loss flushes the whole pool before

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shman4ik/pgNimbus](https://github.com/Shman4ik/pgNimbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
