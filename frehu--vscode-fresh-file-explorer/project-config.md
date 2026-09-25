---
trigger: always_on
description: **Optimize for end-of-conversation satisfaction over next-reply satisfaction** Drop pleasantries. Initial pushback is better than starting the wrong way and going in circles. NEVER brush over concerns you spot, even if unrelated to the issue at hand. You MUST list any additional findings with your next reply.
---

# Fresh File Explorer - Agent Maintenance Guide

## Approach to Problems

**Optimize for end-of-conversation satisfaction over next-reply satisfaction** Drop pleasantries. Initial pushback is better than starting the wrong way and going in circles. NEVER brush over concerns you spot, even if unrelated to the issue at hand. You MUST list any additional findings with your next reply.

**Iteration is fast.** Before diving into long reasoning chains or vscode sources: make a quick attempt to produce better diagnostics and ask the developer to run it. They will report back within the same conversation. This is almost always faster than trying to reason from first principles about VS Code internals.

**When something is unclear or behaving unexpectedly:**
1. Run the git commands the extension runs, inside the repo used to reproduce the problem. Don't assume their output. A wrong assumption is often the problem.
2. Add targeted logging and ask the developer to reproduce and share the output
3. If nothing is working as expected, look at the relevant VS Code source (cloned for you in `../vscode`)

**Unit-testable logic:** Create pure functions. A large part of the functionality relies on hard-to-automate scenarios - specific workspace setups or git history being a certain way. Make sure it at least operates on testable inputs and outputs.

## Non-Obvious Architecture

### Refresh Hierarchy — Use the Cheapest One That's Correct

From cheapest to most expensive:

1. **`refreshTreeOnly()`** — Re-renders from cached data. No git. Use for filter/display changes.
2. **`refreshPending(targetRepoPaths?)`** — Re-runs `git status` only, rebuilds from cached historical baseline. Use when working tree changes (file save, stage, discard). Supports targeting specific repos.
3. **`refresh({ targetRepoPaths? })`** — Re-runs git log for affected repos. Skips repo discovery. Supports `preserveHistoricalCache: true` (e.g. time window switch) and `targetRepoPaths` to scope to specific repos.
4. **`hardRefresh()`** — Clears everything including repo discovery. Only needed when the repo list may have changed (startup, refresh button).

**Historical data is cached across the max configured time window.** Switching time windows almost never requires a git call — `setTimeWindow()` checks whether the in-flight or cached data already covers the new window and falls back to `refreshTreeOnly()` if so. The incremental load fires threshold callbacks so smaller windows get data first.

### Manual Scan — Dormancy Is a Gate, Not a Mode Switch

`freshFileExplorer.freshFileLoading: "manual"` makes the fresh-files scan wait for
an explicit Refresh (an enum, not a boolean, so a future fully-manual mode needs no
settings migration). This setting is meant for users who care about startup impact,
or those who simply don't care about the namesake feature of the fresh file explorer.
Implemented as one predicate (`isScanDormant`) checked at two chokepoints
— `kickOffLoad()` and `refreshPending()`, both in `LoadOrchestrator` — rather
than at every call site that triggers a refresh. Add new automatic refresh triggers *above* those chokepoints and
they're covered for free.

Two rules keep it from breaking neighbouring features:

- **Repo discovery still runs while dormant.** `ensureReposDiscovered()` runs Phase 1
  of the load on its own (deduplicated against a concurrent full load in `RepoDiscovery`) and
  fires `onReposReady` without touching git log. Branch compare, auto-follow and the
  blame heatmap all resolve repos through it, so none of them go dark. It's called
  from activation, not just from the first tree render — the view may be hidden.
- **Explicit consumers arm; ambient ones stay gated.** Anything the user directly
  asked for — quick pick, Code Telescope finder, fresh-files search, the stonks panel,
  the heatmap toggle, filter-by-author/commit — routes through `ensureDataLoaded()`,
  which calls `armScan()`. Silently returning empty would look like a bug. Arming is
  sticky for the session.

  The test is who initiated the *call*, not whether the feature was opted into. The
  heatmap sits on both sides of that line: `handleToggleHeatmap` arms, because a user
  clicked it, while `HeatmapDecorationProvider.provideFileDecoration` checks
  `isScanDormant` and bails, because VS Code polls it for every visible file right
  after activation. Without that second guard, manual scan is a no-op for anyone with
  `heatmap.enabled` on — it would arm immediately after activation with no user action.

  Display-only commands (time window, grouping, sort order) deliberately do *not*
  arm; they'd be a surprising trigger for a scan, and the dormant tree already says
  it hasn't scanned.

`dataLoaded` stays false while dormant, so `enterDormant()` clears
`ContextKeys.LOADING` explicitly — `package.json` `when` clauses depend on it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreHu/vscode-fresh-file-explorer](https://github.com/FreHu/vscode-fresh-file-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
