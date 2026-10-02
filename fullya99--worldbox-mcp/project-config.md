---
trigger: always_on
description: Context for AI coding sessions in this repo. Loaded automatically, so it stays short: only what
---

# CLAUDE.md

Context for AI coding sessions in this repo. Loaded automatically, so it stays short: only what
must be true every time. Everything else lives in `docs/`, which is the published site and the
right place for anything a human contributor also needs.

<!-- style-redaction
prose: en
code: en
commentaires: en
interface: en
git: en
-->

## What this is

`worldbox-mcp` lets any MCP client drive the live game [WorldBox](https://www.superworldbox.com/)
(Unity 2022.3.60f1, Mono). Two pieces shipped from this monorepo:

- **`mod/`**, a BepInEx 5 plugin (net462) injected into the game, exposing a token-authenticated
  HTTP API on `127.0.0.1:8723` and reaching game internals purely through reflection.
- **`server/`**, a Python 3.11+ MCP server on PyPI that proxies tool calls to that API.

<!-- gen-docs:begin total -->29<!-- gen-docs:end total --> tools across six categories. A multi-agent session layer (roles, permissions, fog of war, turn
order, message bus) activates when `BepInEx/config/WorldBoxBridge.agents.json` exists, otherwise
the bridge runs single-tenant.

## Status

- **Latest released**: `v0.6.0`, 2026-09-05, on PyPI and on the GitHub Release with the mod ZIP
  CI attaches by itself. Not yet verified against a live game, and neither was `v0.5.0`, so two
  releases stand on static evidence alone, see [compatibility](docs/compatibility.md).
- **`main` is ahead of it by a feature**, so what PyPI serves is not what the tree does. 0.7.0
  waits in release-please's PR with the two concurrency bounds of #68 and the backstop fix #70
  that made the first one safe. Whoever cuts it makes a third release on static evidence unless
  the live pass happens first.
- **Breaking in 0.5.0**: a `FactionPlayer` agent can no longer call `invoke_power` and gets
  `PERMISSION_DENIED`. God powers are map-wide, so they carry the same gate as `paint_tile`.
  Creature placement stays available through `spawn`.
- **Careful**: the released DLLs for 0.3.0 to 0.3.3 do not load at all. If someone reports a dead
  mod on those versions, that is why, and `LogOutput.log` looks perfectly normal in that state.
- `main` is the shipping branch. release-please keeps a release PR open as commits land.

## Start here

1. `TODOS.md`, the "Pick up here" block first. That is the anchor after a `/clear`.
2. This file, for what is always true.
3. `docs/` for depth: [architecture](docs/architecture.md) for the layout and the request flow,
   [game-api-notes](docs/game-api-notes.md) for the reflection traps, [development](docs/development.md)
   for build, deploy, diagnostics and the release process, [command-reference](docs/command-reference.md)
   for the tool surface, [multi-agent](docs/multi-agent.md) for the session layer.

If an `archives/` directory exists, do not read it when picking up work. It is stale by
construction and only kept for history.

## Conventions

- **Commits**: [Conventional Commits](https://www.conventionalcommits.org/), in English.
  release-please reads them to bump SemVer and write the changelog. **A change that ships no
  code takes a type that does not bump**: `ci:` for workflows, `docs:` for prose, `chore:` for
  tooling and lockfiles. Reaching for `fix:` there opens a release PR for a version nobody can
  install anything new from, which is how #66 came to propose 0.6.1 for a corrected sample
  response and a `gen-docs` check. The rule used to name only `ci:`, and that was too narrow.
- **Merge PRs with a merge commit, never a squash.** The repo takes the PR title as the squash
  subject, so squashing a PR titled `deps: ...` hides the `feat:` commits inside it and the minor
  bump is silently skipped. The one exception is release-please's own PR, which is squashed.
- **Give the merge commit a body that is not a Conventional Commit.** `gh pr merge --merge`
  defaults the body to the PR title, and PR titles here are usually Conventional Commits, so
  release-please counts the work twice and the changelog ships duplicated. Pass `--body` with a
  short review note. See [development.md](docs/development.md).
- **Everything is written in English**, including prose, comments and commits.
- **C#**: nullable enabled, warnings as errors, formatted by csharpier.
- **Python**: ruff for lint and format, `mypy --strict`. Annotate everything, `Any` only at the
  MCP boundary.

## Rules that are easy to break

- **Every Unity API call is marshalled onto the main thread**, through
  `MainThreadDispatcher.RunOnMainThreadAsync` for a single call or
  `RunPerFrameOnMainThreadAsync` for one that repeats per frame. Anything else corrupts game
  state without an error. A command may legitimately report `RequiresMainThread => false` and
  marshal only the game call, which is what keeps blocking I/O out of a frame, see
  `LoadWorldCommand`. And `true` marshals the command's first thread, not its whole body: an
  `await` inside one escapes the dispatcher's deadline and per-frame cap. The remarks on
  `ICommand.RequiresMainThread` carry the detail.
- **No `System.ValueTuple`** in a signature, a field type or a dictionary key. It is not always
  loadable under Unity Mono on net462. Use a `readonly struct`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fullya99/worldbox-mcp](https://github.com/fullya99/worldbox-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
