---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

memo is a local-first semantic memory: MLX embeddings + sqlite-vec hybrid
(vec + BM25) search, reranking, a knowledge graph, temporal reasoning, an MCP
server, and a CLI. Single-user, runs offline. PyPI distribution name is `mlx-memo`; CLI binary is `memo`; MCP server binary is `memo-mcp`.

## Dev commands

```bash
# Tests (no MLX needed — MLX tests skip automatically on non-Apple Silicon)
uv run --no-sync pytest tests/                          # full suite
uv run --no-sync pytest tests/test_foo.py::test_bar -v  # single test

# Type checking
uv run --no-sync mypy src/memo/

# Lint + format
uv run --no-sync ruff check src/ && uv run --no-sync ruff format src/

# Run the CLI
uv run --no-sync memo <cmd>

# Validate MEMO_* flags (typos, unknown vars)
uv run --no-sync memo config validate
```

## Working tree is shared (read before any git op)

This repo is frequently worked on by **concurrent agent sessions sharing one
working tree and HEAD**. Their commits land on whatever branch is checked out,
master advances underneath you, and a `git checkout` in another session moves
*your* HEAD too. So:

- **Never `git add -A` / `git commit -a` / `ruff format src/`** — they sweep in
  other sessions' in-flight files. Stage explicit paths only:
  `git add src/memo/foo.py tests/test_foo.py`. Lint/format only your files.
- **Check `git reflog` + `git status` before any `reset`/`checkout`** — you may
  un-reference a concurrent commit or move a shared HEAD.
- **Cut releases from an isolated worktree**, never `git checkout master` in the
  shared tree: `git worktree add --detach /tmp/rel origin/master`, cherry-pick
  your commit, bump versions + CHANGELOG, commit, `git push origin HEAD:master`,
  tag, `git worktree remove --force`. Check `git ls-remote --tags origin vX.Y.Z`
  first — a concurrent session may have taken your version number.
- Nothing is lost on entanglement (commits live in `git reflog`, uncommitted
  work stays in the tree). Rebuild a clean branch with
  `git checkout <branch> -- <only your files>` (zsh does NOT word-split `$VAR` —
  list paths explicitly).

## Architecture

**`Memory` facade** (`src/memo/memory/facade.py`) multiply-inherits thirteen operation mixins — `_WriteOpsMixin`, `_UpdateOpsMixin`, `_DeleteOpsMixin`, `_SearchOpsMixin`, `_SearchScoringMixin`, `_AskOpsMixin`, `_ChatAskOpsMixin`, `_RerankOpsMixin`, `_RepoOpsMixin`, `_MaintainOpsMixin`, `_ConsolidateOpsMixin`, `_ReplayOpsMixin`, `_SecretOpsMixin` — each in their own `src/memo/memory/<op>_ops.py` file. Module-level constants, prompts, and pure helpers are in `src/memo/memory/record.py`. Never import from a mixin directly; always go through `Memory`.

**MCP server** (`src/memo/server.py`) registers tools via `build_server()`. Each domain is a `server_<domain>.py` module that exports `register(server, memory)` — called once in `build_server()`. Adding a new MCP tool = create `src/memo/server_<domain>.py` + add one `register` call in `server.py`. The `_MaintainOpsMixin` method is directly callable from tests via `mock_memory.<method>()`.

**Storage** (`src/memo/store/`) is a subpackage. `VecStore` in `queries.py` is the primary interface: one sqlite-vec DB file, thread-local connections (required for FastMCP HTTP worker threadpool). Writes use `_tx()` (`BEGIN IMMEDIATE`); vectors are packed float32 blobs; WAL mode + `busy_timeout`.

**CLI** is in `src/memo/cli.py` (entry-point wiring only) + `src/memo/cli_<domain>.py` files. Each domain file exports a Click command or group imported and registered in `cli.py`.

**Flags** (`src/memo/flags.py`) is the single registry for all `MEMO_*` env vars — it aggregates `FlagSpec`s defined in the per-domain `flags_<group>.py` modules (`flags_recall/search/behavior/capture/ingest/misc.py`, base types in `flags_base.py`). Add a new flag in the matching `flags_<group>.py`. Use `flag_bool/int/float/str(name)` — never `os.environ.get("MEMO_...")` inline. `memo config validate` parses every set flag.

**Cache** (`src/memo/cache.py`): query embeddings use Memo's native
thread-safe LRU cache when `MEMO_QUERY_CACHE_SIZE > 0`. The stable path never
loads a cache implementation from another package.

**Contracts and replay URIs** (`src/memo/contracts.py`,
`src/memo/memory/replay_ops.py`): Memo owns its request/result contracts and
`memo://` parser. Legacy field names are migration aliases only.

**Sync — two explicit tiers** (`src/memo/sync_git.py`, `src/memo/identity.py`):
- **LOCAL (intra-machine):** all sessions on a Mac share one `data_dir`/`memvec.db`
  (WAL, thread-local conns), so a save/capture is visible to sibling sessions on
  their next recall **with no git**. `sync_tier(cfg)` → `"local"` when no git
  remote is configured.
- **REMOTE (inter-machine):** the git `memo-sync` remote is the only cross-machine
  channel. `sync_tier(cfg)` → `"remote"`. ONE owner per machine: `sync_once()`
  takes an `flock` on `state_dir/.sync.lock` (concurrent same-machine sessions
  skip — their writes ride the lock holder's push) and does
  **pull-rebase-before-push** so an advanced remote rebases instead of rejecting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jagoff/memo](https://github.com/jagoff/memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
