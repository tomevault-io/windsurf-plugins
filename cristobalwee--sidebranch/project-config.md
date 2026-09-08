---
trigger: always_on
description: Canonical reference for any AI agent (or human) working in this repository.
---

# AGENTS.md

Canonical reference for any AI agent (or human) working in this repository.
`CLAUDE.md` just points here — keep this the single source of truth rather
than letting two docs drift.

## What this is

sidebranch: a local, zero-dependency sidecar for reviewing PRs from inside a
running app — pick a branch from an in-page widget, it builds in an isolated
`git worktree` on its own dev server, your own working tree is never
touched. See `README.md` for the user-facing pitch and `SECURITY.md` for the
full threat model. This file is about building/changing the tool itself.

## Non-negotiables — read before touching anything

1. **Zero runtime dependencies.** `package.json`'s `dependencies` must stay
   `{}`. Everything is Node builtins (`http`, `fs`, `child_process`, `net`,
   `crypto`, `events`) and vanilla browser APIs. Don't reach for a package
   to solve a problem here — solve it with what's already available, or
   accept a rougher edge.
2. **The security invariants in `SECURITY.md` are not configurable and not
   negotiable.** Loopback-only bind, peer-address check, Host header
   allowlist, Origin allowlist, bearer token on every `/api/*` call, no
   shell ever (`execFile`/`spawn` with argument arrays only), no filesystem
   routing. If a bug or feature request seems to require relaxing one of
   these, it doesn't — find a different way, or say so explicitly rather
   than quietly loosening a check.
3. **Never run a mutating git command against the user's primary working
   tree.** Reads only (`status`, `branch --list`, `for-each-ref`, etc.).
   All mutation happens inside `~/.sidebranch/projects/*/panes/*`
   worktrees, which are append-only from the tool's perspective — a dirty
   pane fails closed (`EDIRTY`) rather than being silently reset, unless
   the caller explicitly opts into `discard`.
4. **`repoRoot` resolves from `process.cwd()`, exactly like git.** There is
   no `--repo` flag and no safety check. Run `sidebranch start` from
   *inside this repo* and it will happily start managing itself (this
   repo is a git repo like any other) — which is occasionally what you
   want and usually isn't. Always confirm the `repo` path the startup
   banner prints matches what you expect before trusting anything that
   follows.
5. You **MUST NOT** write any code comments
6. You **MUST** strive to write code with **low cyclomatic complexity**

## Architecture

```
bin/sidebranch.js  -> src/cli.js         init | start | stop | clean | doctor
src/daemonfile.js                         per-repo {pid,port} record: lets stop signal a daemon
                                           and start/clean/doctor detect one. Liveness is proven
                                           twice (pid + /healthz) — never trust the pid alone.
src/config.js                            .sidebranch.json load/normalize, data-dir resolution.
                                           normalizeEnv() sanitizes the `env` map; RESERVED_ENV
                                           lists the vars sidebranch injects and config can't set.
src/gitops.js                             every git invocation (execFile only; refs validated twice)
src/manager.js                            pane orchestration: worktree -> checkout -> conditional
                                           install -> (re)start dev server. Mutations serialized
                                           through Manager.run() so concurrent widget clicks can't
                                           interleave git operations on the same pane.
src/processes.js                          DevServer: spawns/supervises one dev server per pane,
                                           confirms readiness by polling the port (never parses
                                           stdout — some servers pick a different port than asked)
src/install.js                            lockfile hashing (skip reinstall when unchanged) + install.
                                           Also hosts pushRing/tailText, the small ring-buffer +
                                           collapsed-tail helpers DevServer and Manager both reuse to
                                           bake a snippet of captured output into failure messages.
src/daemon.js                             HTTP server: loopback/Host/Origin gate, bearer auth,
                                           routes, SSE event stream (/api/events), GET
                                           /api/pane/:id/log for a pane's full install+server output,
                                           GET /handshake (unauthenticated token bootstrap for the
                                           extension; API_VERSION lives here)
src/security.js                           the trust boundary every check in daemon.js/gitops.js
                                           relies on
src/assets/widget-core.js                 in-page pill/toolbar; runs INSIDE the consumer app's
                                           page, shadow DOM (mode:"closed"). Defines
                                           globalThis.__sidebranchStart({token, port}) and does
                                           nothing on load — a boot file calls it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cristobalwee/sidebranch](https://github.com/cristobalwee/sidebranch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
