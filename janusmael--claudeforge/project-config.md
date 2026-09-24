---
trigger: always_on
description: > Audience: an agent (Claude or otherwise) returning to ClaudeForge cold.
---

# AGENTS.md — operational rules for LLM contributors

> Audience: an agent (Claude or otherwise) returning to ClaudeForge cold.
> Purpose: surface cross-file contracts that aren't visible from a single-file
> read, so you don't break invariants you can't see.
> Methodology rationale: see [`AGENT-ONBOARDING.md`](./AGENT-ONBOARDING.md).
> Narrative architecture and prose context live in [`CLAUDE.md`](./CLAUDE.md).

This file is **fact-shaped**: every claim cites a file path, function name, or
test name. If a fact here is wrong, grep for the identifier — code drift will
surface as a missing or relocated symbol, not as silently-stale prose.

Two specific anti-patterns this file refuses on principle:

- **No hardcoded source-line numbers** (`Foo.cs:245`). They drift on every
  refactor and turn the doc into a liar. Cite the file, the type, the method,
  or `nameof()` — let `grep` do the locating.
- **No timestamps in prose** ("Reported 2026-05-13", "shipped 2026-05-07").
  `git log` and `git blame` are the authoritative source for when a thing
  happened; carrying the date in prose adds maintenance debt with no
  corresponding benefit.

---

## 1. Hard invariants

| Invariant | Failure signature if you break it | Canonical source |
|-----------|-----------------------------------|------------------|
| **Compound editors must use the force-fire `MarkModified()` pattern**, never bare `IsModified = true`. CommunityToolkit.Mvvm's `[ObservableProperty]` setter elides equal assignments, so a bare assignment when the flag is already `true` (e.g. after `LoadFromLayered` set it for an already-populated scope) is a no-op and the live-write / Save-button-enable chain never runs. | User edits or removes an item on a loaded compound editor → Save button stays disabled. Or: user edits a property whose backing field was already populated → no live-write to disk. | Helper: `MarkModified()` in `McpServersEditorViewModel`, mirrored in `HooksEditorViewModel`, `PermissionsEditorViewModel`, `EnabledPluginsEditorViewModel`, `MarketplacesEditorViewModel`. Sidecar: [`src/ClaudeForge/ViewModels/Editors/AGENTS.md`](./src/ClaudeForge/ViewModels/Editors/AGENTS.md). |
| **`ConfigScope` is a struct, and two of its semantics are invisible to the compiler.** (1) `default(ConfigScope)` MUST stay `Managed` — it is backed by a single ordinal for exactly this reason, and a dozen editors declare `private ConfigScope _lastScope;` with no initialiser. (2) `ToString()` MUST keep returning the old enum member names; it is consumed as data, not displayed. Also: it cannot be a default parameter value or a `case` label — use an overload and `when` guards. | A richer struct shape (e.g. a record of `Id`/`Priority`/`DisplayName`/`IsReadOnly`) compiles, passes **2,791 of 2,792 tests**, and silently changes what an uninitialised scope means. Breaking `ToString()` silently breaks scope brushes, tooltips and labels across the editors. | Type: `src/AgentForge.Core/Settings/ConfigScope.cs`. Guard: `tests/AgentForge.Core.Tests/Settings/ConfigScopeTests.cs`. Sidecar: [`src/AgentForge.Core/Settings/AGENTS.md`](./src/AgentForge.Core/Settings/AGENTS.md). |
| **A guard's comment is not evidence the guard covers what it says.** `_reloadPending` claimed to "prevent concurrent calls to `LoadAllWorkspacesAsync`"; it guards `ReloadCoreAsync`, one of three callers. Serialisation now lives in `LoadAllWorkspacesAsync` itself. When a concurrency invariant matters, put it in the method performing the destructive change — not in each caller. | A use-after-dispose race (`ObjectDisposedException` from the nav-tree build) survived for years because the test written to cover it named the wrong guard AND could not fail. `OpenProjectAsync` sets `IsLoading` without checking it and awaits a dialog first, so a file-watcher reload starts underneath it. | Type: `MainWindowViewModel.LoadAllWorkspacesAsync` — read its remarks before changing it; **serialise, never coalesce** (a joined load never opens the new `ProjectRoot`) and **chain, never lock** (the load path can re-enter). Guard: `ReloadHardeningTests`. |
| **`Session.Dispatch(async () => …)` in a headless test CANNOT FAIL.** It binds `Dispatch<T>(Func<T>)` with `T = Task`, yielding `Task<Task>` whose inner task is never awaited. Return a value from the lambda so it binds `Dispatch<T>(Func<Task<T>>)`, then canary with `Assert.Fail`. Non-`async` lambdas are fine — they bind `Dispatch(Action, ct)`. | 19 tests were inert from the day they were written and hid **two real defects**, including a data-loss path where an unparseable config was swapped into memory and then saved over. | Guards: `tests/ClaudeForge.Tests/Headless/*`. Working pattern: `ExportArchiveTests`. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JanusMael/ClaudeForge](https://github.com/JanusMael/ClaudeForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
