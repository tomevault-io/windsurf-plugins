---
trigger: always_on
description: An agentic research workspace: a dockable-pane UI (Vue) over a Python sidecar that drives
---

# Panel — working in this repo

An agentic research workspace: a dockable-pane UI (Vue) over a Python sidecar that drives
agent backends and runs Modules. See [README.md](./README.md) for the vision and
[docs/](./docs/) for the design.

## Two runtimes

This is a polyglot monorepo. **The frontend and the backend are separate processes and
separate package managers.** Getting this wrong is the most common mistake here.

| | Path | Manager | Never use |
|---|---|---|---|
| Web app | `apps/web` | `pnpm` | `npm`, `yarn` |
| Sidecar | `apps/server` | `uv` | `pip`, `poetry`, a manually-activated venv |

```sh
pnpm install                 # workspace root — installs apps/web
uv sync                      # workspace root — installs apps/server + dev group
pnpm dev                     # runs both: Vite on :5173, sidecar on :8787
```

`pnpm dev` is `scripts/dev.mjs`, which runs both and **reaps the whole process tree on
exit**. It replaced `run-p` for that second half: `npm-run-all2` tree-kills only on task
failure and installs no signal handlers, so on Windows a Ctrl-C left twelve descendants
running. To run one side alone:

```sh
pnpm dev:web
pnpm dev:api
# if `uv run --directory` misbehaves on your platform:
cd apps/server && uv run uvicorn panel_server.app:app --port 8787
```

Vite proxies `/api` to `127.0.0.1:8787`, so the browser only ever talks to :5173.

**`pnpm dev` offers dev Modules; `pnpm start` does not.** `dev:api` loads
`apps/server/dev.env`, which sets `PANEL_DEV_MODULES=1` and so puts `tally` in the
catalogue. `pnpm start` builds the web app, serves it with `vite preview` on :4173 (proxying
`/api` the same way), and starts the sidecar without that file. It is what a tester runs.
The test suite sets the flag itself in `conftest.py`, because `MODULES` is built at import.

**The sidecar runs without `--reload`, and on Windows it must.** uvicorn picks its event
loop with `asyncio_loop_factory(use_subprocess)`, where `use_subprocess` means uvicorn's
own reload supervisor — and on Windows that choice costs *the application* the loop that
can spawn subprocesses:

| | loop | can a backend spawn a CLI? |
|---|---|---|
| `--reload` | `SelectorEventLoop` | no |
| plain | `ProactorEventLoop` | yes |

So `pnpm dev` with `--reload` could never run Claude Code, and would fail Codex the same
way. The symptom names neither the loop nor the reason: `CLIConnectionError: Failed to
start Claude Code:` with **nothing after the colon**, because the message is
`f"...: {e}"` and a bare `NotImplementedError` stringifies to `""`. An empty tail there is
the signature — the CLI is installed and fine.

**A jupyter kernel is a subprocess too, so it depends on the same loop** — and it also wants
something `ProactorEventLoop` does not have, `add_reader`, which zmq uses. Measured rather
than assumed: pyzmq registers a selector thread through tornado, prints a `RuntimeWarning`
saying so, and works. That warning in the sidecar's output is expected and is not the thing
to chase.

Editing Python therefore needs a manual restart. An external watcher would restore that,
but the one tried (`watchfiles` wrapping uvicorn) killed the whole console twice while
handling a change, taking the terminal with it, so it is not in place. Anything attempted
here should be started in its own console and proven not to signal the one it came from.

Verify both are up: `curl http://localhost:5173/api/health`.

### When the app looks broken before you have changed anything

```sh
pnpm dev:doctor
```

Reports each dev port as `free`, `healthy`, or `STALE`, and prints the command to clear a
stale one. The third state is the one worth knowing about: kill `pnpm dev` in a way that
skips the reaping — Task Manager, a force-stop, a closed terminal on some shells — and
`uvicorn --reload`'s supervisor can outlive its worker while keeping :8787 bound. The port
accepts connections and answers nothing, so the next run presents as a broken server
rather than as a leftover. `pnpm dev:doctor` tells the two apart in one command; a bare
`netstat` cannot.

Note Vite binds **IPv6** loopback. `http://127.0.0.1:5173` is refused while the server is
perfectly healthy, and `netstat -ano -p TCP` hides it — that flag means IPv4 only.

A fourth state exists that `dev:doctor` reports as `healthy`: a **surviving worker whose
supervisor was killed**. Kill the supervisor's pid alone and its `multiprocessing` child
lives on holding the inherited socket, so the port answers correctly while running
whatever code the dead tree started with. The recorded socket owner is then a pid that no
longer exists, which is the giveaway — and the worker is hard to find by name, since its
command line is `spawn_main(parent_pid=...)` and mentions neither `panel` nor the port.
`taskkill /T` on the supervisor avoids creating it; `Get-NetTCPConnection -LocalPort 8787`
plus a `tasklist` on the owner it names finds it afterwards.

## Layout

```
apps/web        Vue 3 + Vite + dockview + Tailwind + Pinia
apps/server     FastAPI sidecar — Runs, backends, Modules, DAL, custom Panes
packages/       not yet created; see docs/repo_structure.md §5 for the trigger
docs/           architecture and design; read before changing structure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greentfrapp/panel](https://github.com/greentfrapp/panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
