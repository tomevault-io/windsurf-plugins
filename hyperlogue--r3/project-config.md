---
trigger: always_on
description: validates against it. The global sqlite is the only process-wide singleton;
---

# r3 — Review. Revise. Resolve.

A **local-first review tool for AI-generated code and docs**. A long-running
per-user daemon on localhost owns review + feedback state for all your repos;
reviews are created from the CLI/agent, and you review the commits, diffs, and raw
files they capture in the browser and leave line/quote-anchored **feedback**; an AI
agent (or you) **replies** by id and the decision shows up **live** over SSE. The
daemon, CLI, and SPA ship as one self-contained binary.

For usage see [`README.md`](README.md). This file is the orientation map for
working _in_ the repo **and the single source of truth for its design** — when a
design decision changes, update the relevant section here.

**Why r3 / prior art.** When an AI agent writes code or docs you want to read the
result, mark the exact spots you care about, hand those notes back, and watch the
agent react in place — without copy-pasting transcripts. The MIT-licensed **difit**
and **diffx** informed the design; r3's delta from both: a **persisted review +
feedback/reply model**, **raw-file** (not just diff) reviews, headless CLI creation
with queryable session/meta, an **agent re-anchor API** to keep feedback from
orphaning, and a structured **reply/watch protocol** that round-trips live into
the UI.

## Architecture

The server is **authoritative**; there are **three clients of one HTTP/JSON API**
— the browser (SPA), the CLI, and the agent (through the CLI). Because the CLI and
agent are first-class, **the HTTP/JSON contract in `shared/types.ts` is the
product**, not an implementation detail of the React client.

```
          ┌── browser (SPA)  ─ fetch + SSE ──┐
agent ── CLI (thin HTTP client) ─ HTTP ───────┼──►  daemon (Hono + bun:sqlite)
          └── you at the terminal ────────────┘        one per user, one port,
                                                        one global store
```

- **One per-user daemon** spans every repo, on a stable port (default 8791),
  behind one origin. It's spawned **lazily** on the first CLI call (or by opening
  the browser) à la the tmux server — nothing to start by hand — and announces
  itself in `$XDG_RUNTIME_DIR/r3/daemon.json` so the CLI finds it with zero config.
- **The CLI is the single entry point and the binary.** `cli/index.ts` is a thin
  HTTP client — every command is one HTTP call; it never writes sqlite directly
  (single writer, the server stays authoritative). A hidden `__daemon` subcommand
  re-execs the same script/binary to _serve_; `ensureServer()`
  discovers-or-lazily-spawns the daemon.
- **Reviews live in one global sqlite** (`$XDG_STATE_HOME/r3/r3.sqlite`) keyed by a
  **projects registry**, not per-repo files. A project's identity is its **shared
  git object store** (`realpath(git rev-parse --git-common-dir)`), so all worktrees
  of one clone are one project and a `cp -r` copy is a distinct empty one.
- **The server core is de-globalized** into a per-request `Repo` context
  (`server/repo.ts`): `{ repoId, commonDir, worktreePath, descriptor, stale, git(),
gitText(), safePath() }`. `git()` runs with `cwd = worktreePath`; `safePath()`
  validates against it. The global sqlite is the only process-wide singleton;
  everything else is per-Repo.
- **The daemon is repo-agnostic** — it holds no ambient "default repo". Each
  request resolves its `Repo` fresh, most-specific first: a `?review=<id>` (the row
  carries its repo), the CLI's `x-r3-repo` header (computed per call from the CLI's
  own checkout), or the browser's `?repo=<id>` selector. A request that names none
  gets `null` → `400 "no repo context"`; the CLI refuses a repo-scoped command
  (e.g. `r3 create`) run outside a git repo rather than letting it reach the daemon.
- **Freshness + live updates** flow one way to the clients: a file watcher
  (`server/watcher.ts`) watches only the files open reviews reference and pushes
  `file-changed`; every review/feedback/reply write bumps `review.updated_at` and
  broadcasts over SSE (`server/sse.ts`). The SPA invalidates its TanStack Query
  cache on the matching event.

A **worktree** shares its clone's common-dir, so it's the _same_ project — but it
has its own working tree, index, and HEAD, so a review records a `worktree`
descriptor `{ name, branch, pathHint }` and runs its git ops in the exact worktree
it was created in. Resolution matches `worktree.name` (then branch) against live
`git worktree list`, so `git worktree move` auto-resolves; a removed worktree falls
back to the primary for immutable reviews and flags live ones `stale`. A moved
_repo_ is a one-row `UPDATE repos SET common_dir=…` (`r3 repo relink`) — reviews
reference the immutable `repo_id`, never a path. A `cp -r` copy has a new
common-dir ⇒ a distinct empty project (identity lives only in the store, so there
is no `.r3/id` marker to confuse a move with a copy).

## Module layout

Start at `shared/types.ts` — **the HTTP contract** all three clients agree on —
then `server/index.ts` (daemon entry, routes, guards), `server/repo.ts` (the Repo
context), and `cli/index.ts` (the binary and the agent's entry point).

```
server/          Hono daemon + bun:sqlite global store
  index.ts       startDaemon(): HTTP/JSON API + SPA serving + host/token guards

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyperlogue/r3](https://github.com/hyperlogue/r3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
