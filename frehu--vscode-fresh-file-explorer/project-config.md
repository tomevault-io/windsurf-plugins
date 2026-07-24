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

**`refreshEpoch`** is incremented by `refresh()` and `hardRefresh()`. `updateFreshFiles()` checks it at each async boundary and throws `RefreshCancelledError` if a newer refresh started. Always preserve this check when adding async work to the load path.

## `files.exclude` is Per-Node, Not Global

The tree honors VS Code's `files.exclude` (`freshFileExplorer.respectFilesExclude`, default on). The non-obvious part: a glob is evaluated **relative to the workspace folder of the node it's rendered under**, so the *same* absolute file can be hidden under one root and shown under another. The motivating case (issue #3): a repo root added alongside its own `backend/` subfolder, with the root excluding `backend` — `backend/app.js` must vanish under the root node yet remain under the backend node. A single global "display map" cannot express this; don't try to filter `_freshFiles` once and reuse it everywhere.

- Pure glob matching: [filesExcludeMatcher.ts](src/fresh-files/filesExcludeMatcher.ts) — a faithful port of VS Code's own `vs/base/common/glob.ts` engine (not minimatch). A bare `backend` matches the path *and its ancestor prefixes* (mirrors the Explorer pruning the `backend` dir node); bare patterns stay root-anchored.
- Per-node application: [FilesExcludeFilter](src/fresh-files/filesExcludeFilter.ts). `isExcludedUnder(path, folder)` is the per-node check used by `buildTree`/`buildFlatList`/`buildRepoView`. `isExcludedByOwner` / `filterByOwner` handle the flat lenses (group-by-author/commit, search) that have no node context.
- Both are pure/unit-tested; the provider holds only wiring + a compiled-glob cache invalidated on config change. `when`-clause (sibling) excludes are unsupported.

## Path Handling

- Git uses forward slashes; Windows uses backslashes. We try to stick to normalized paths `/`.
- Use and define branded types for different path variants
- `asAbsolutePath()` calls `normalizePath()` internally

## Git Command Execution

Always use **`execGitWithArgs(args[], cwd)`** — uses `spawn()` with an argument array, no injection risk. For large outputs, stream instead of buffering: `streamGitLogNameStatus` in `gitOperations.ts`, or the diff-search parser's own `streamGitDiffOutput` (local to `diffSearchParser.ts`).

## Webview Message Protocol

The host↔webview boundary is a `postMessage` contract typed by discriminated
unions in `src/webview/messages.ts` (compiled by both tsconfigs). Every host
panel sends through a typed `_post(msg: XToWebview)` wrapper and types its

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreHu/vscode-fresh-file-explorer](https://github.com/FreHu/vscode-fresh-file-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
