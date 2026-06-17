---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md / AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

(If you are Codex CLI or other AI coding agents, this is loaded via AGENTS.md.)

## Build & Development Commands

```bash
pnpm build              # Build extension + webview
pnpm build:prod         # Production build (minified, no sourcemaps)
pnpm build:ext          # Build extension only (esbuild)
pnpm build:webview      # Build webview only (Vite)
pnpm watch              # Watch mode for both (uses concurrently)
pnpm lint               # ESLint (flat config) over the whole repo
pnpm typecheck          # TypeScript type checking (tsc --noEmit)
pnpm test               # Run unit tests (Vitest, run mode)
pnpm generate-test-repo # Generate deterministic test repo at test-repo/
pnpm generate-submodule-repos # Generate test repos with submodules
pnpm ext:package        # Create .vsix package
pnpm ext:publish        # Publish to VS Code Marketplace (vsce) + Open VSX (ovsx)
```

Run a single test file or pattern with Vitest directly:

```bash
pnpm vitest run path/to/file.test.ts   # one file
pnpm vitest run -t "test name substring" # filter by test name
pnpm vitest                              # watch mode
```

To debug: use VS Code launch configs "Run Extension" or "Run Extension (Watch)" in `.vscode/launch.json`.

## Architecture

VS Code extension with **backend** (Node.js extension host) and **frontend** (React webview), communicating via VS Code's message passing API (`postMessage`/`onDidReceiveMessage`).

```
src/                              # Backend — esbuild → dist/extension.js (CJS, node18)
├── extension.ts                  # Entry point, registers speedyGit.showGraph command
├── ExtensionController.ts        # Orchestrates services, repo discovery, settings
├── WebviewProvider.ts            # Compatibility re-export of webview/WebviewProvider
├── GitShowContentProvider.ts     # git-show:// URI protocol for diffs
├── webview/                      # Backend webview subsystem (refactored from the old ~2400-line WebviewProvider)
│   ├── WebviewProvider.ts        # Thin public facade used by ExtensionController; composes the objects below
│   ├── WebviewPanelHost.ts       # VS Code panel lifecycle, HTML/CSP/nonce, postMessage, visibility
│   ├── WebviewRuntime.ts         # Mutable non-service state: repo path, filters, fetch generation, flags
│   ├── GitServiceRegistry.ts     # Holds repo-bound git services; atomic replacement on repo switch
│   ├── WebviewMessageRouter.ts   # Exhaustive typed RPC dispatch (satisfies RequestHandlerMap)
│   ├── WebviewRequestContext.ts  # Narrow per-request handler API (no provider instance leaks to handlers)
│   ├── PersistedUIStateStore.ts  # Load/save/validate UI state + per-repo table layout (column-width healing)
│   ├── RepoDataLoader.ts         # Initial + deferred data, avatars, submodules; what to fetch and post
│   ├── RefreshCoordinator.ts     # When to load: initial/manual/auto, hidden-panel deferral, loading lifecycle
│   ├── EditorCommandService.ts   # VS Code diff/file/compare editors, worktree folder/reveal, signature help
│   ├── OperationGuard.ts         # In-progress checks (rebase/cherry-pick/revert/merge) → GitError | null
│   └── handlers/                 # Domain RPC handlers, grouped by feature; fetch services from registry at call time
│       ├── handlerUtils.ts       # Shared handler helpers
│       ├── graphDataHandlers.ts  # getCommits/loadMore/getBranches/getCommitDetails/getAuthors/refresh
│       ├── branchHandlers.ts     # checkout/create/rename/delete/merge/fast-forward branch
│       ├── remoteHandlers.ts     # fetch/push/pull, add/edit/remove remote
│       ├── tagHandlers.ts        # create/delete/push tag
│       ├── stashHandlers.ts      # get/apply/pop/drop/create stash
│       ├── historyHandlers.ts    # reset/cherry-pick/revert/rebase + continue/abort, dropCommit
│       ├── signatureHandlers.ts  # presence detection, verification, signature help
│       ├── submoduleHandlers.ts  # submodule ops + switchRepo/displayRepo navigation
│       ├── worktreeHandlers.ts   # list/resolve/add/remove/prune/open/reveal worktree
│       ├── workingTreeHandlers.ts# uncommitted changes, stage/unstage/discard, diff editors
│       ├── compareHandlers.ts    # compareRefs/cancelCompare/openCompareDiff (latest-wins by request id)
│       └── vscodeCommandHandlers.ts # settings, clipboard, openExternal, updatePersistedUIState
├── services/
│   ├── index.ts                  # Barrel export for all services
│   ├── GitExecutor.ts            # Spawns git processes, 30s timeout, returns Result<T, GitError>
│   ├── GitLogService.ts          # Parses git log (null-byte format), branches. Default 500 commits
│   ├── GitDiffService.ts         # Commit details, file changes, file content at revision
│   ├── GitBranchService.ts       # Checkout, create, rename, delete, fast-forward branches
│   ├── GitRemoteService.ts       # Fetch, pull, remote management
│   ├── GitHistoryService.ts      # Rebase, reset operations
│   ├── GitRebaseService.ts       # Interactive rebase with drag-drop reordering
│   ├── GitCherryPickService.ts   # Cherry-pick with conflict handling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [onlineeric/speedy-git-ext](https://github.com/onlineeric/speedy-git-ext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
