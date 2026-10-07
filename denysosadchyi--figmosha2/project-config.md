---
trigger: always_on
description: For any coding agent working in this repo: Codex reads this file directly, Claude Code through
---

# Figmosha 2.0 — agent instructions

For any coding agent working in this repo: Codex reads this file directly, Claude Code through
`CLAUDE.md` (which just imports it). Machine-specific paths and hosts live in `CLAUDE.local.md`
(gitignored) — **read it first if it exists**, whichever agent you are.

Drive Figma by sending JS code through a local bridge that's connected to a custom plugin running inside Figma Desktop.

## How to send code

> **macOS / Linux:** there is no `python` command on a stock Mac — wherever this file says
> `python figmosha.py`, run `./venv/bin/python figmosha.py` (or `python3 figmosha.py`).

```bash
# Preferred — subcommand-style
python figmosha.py exec "return figma.currentPage.name"
python figmosha.py exec --file script.js

# Shorthand (auto-prepends `exec`)
python figmosha.py "return figma.currentPage.name"

# High-level commands (covered below) save tokens for common operations
python figmosha.py text 185:21880 "Привіт"
python figmosha.py variant 185:21883 "Property 1=Default"

# Quick HTTP (no Python needed)
curl -s -X POST http://localhost:8787/exec \
  -H 'Content-Type: application/json' \
  -d '{"code":"return figma.currentPage.id"}'

# Status
curl -s http://localhost:8787/status   # {"plugin_connected": ..., "files": [...], "pending": 0, "abandoned": []}
```

## Multiple files: targets (`-T`) — READ THIS FIRST

The bridge holds **one connection per open Figma file** that is running the plugin, not one globally.

```bash
python figmosha.py targets                      # name / doc id / fileKey / conn / queue (busy, waiting)
curl -s http://localhost:8787/targets

# -T is a per-subcommand flag — it goes AFTER the subcommand, never before it
python figmosha.py exec "return figma.root.name" -T "Component Library"
python figmosha.py tree 123:456 -T "Component Library"
curl -s -X POST http://localhost:8787/exec -d '{"code":"...","target":"Icons"}'
```

`target` matching (`resolve_target` in `bridge.py`): document id (the `doc` column, 6+ chars) →
connection id → exact file name (case-insensitive) → exact `fileKey` → unambiguous substring of the name.

- **Two different files with the same name** (two fresh "Untitled" files, say) → the name is a 409;
  target each by its **document id**: `-T dk2m9q4x`. Each file carries its own id (plugin data
  `figmosha-doc` on the document root, written once on first run), so the bridge never mixes them up,
  and the id survives reconnects and bridge restarts. The `conn` id changes on every reconnect — don't
  store it.

- The name is `figma.root.name` as the plugin reports it, which is often not the name you remember —
  a trailing plural, a rename that never propagated. **Verify with `targets`, don't assume.**
- **`fileKey` targeting does not work for a local dev plugin** — it reports `fileKey: null` (shown as
  `-` in `targets`). A file key is still what `importComponentByKeyAsync` needs; it is just not usable
  as a `-T`. Match by name only.

- **No target + exactly 1 file connected** → routed there (the old default).
- **No target + 2 or more connected** → **HTTP 409** `"N files connected — specify a target"`.
  This is the #1 wasted call. **Always pass `-T`** with the file name from the project profile.
- Ambiguous substring → 409 listing the candidates. Nothing connected → 503.

### Concurrency: the per-file queue

**The bridge queues execs per document.** Requests are multiplexed by request id, and `/exec` waits for
the document's lock before sending — so two callers on the same file (orchestrator + agent, or two
agents) run **one after another, never interleaved**, and callers on *different* files never wait for
each other. The lock belongs to the document, not the connection: one file open in two tabs is one
queue.

**Several agents at once — name yourself and bound your wait:**

```bash
export FIGMOSHA_AGENT=designer            # or --agent designer / -A designer, or "agent" in the JSON
python figmosha.py exec --file build.js -T "Component Library" --queue-timeout 30
```

- `targets` / `/status` show each file's queue: who is running and for how long, who is waiting.
- A reply that waited carries `queued_ms`; the CLI prints `(waited 2.1s in the file's queue)`.
- `--queue-timeout` (JSON `queue_timeout`, default = `--timeout`) caps the wait. When it runs out the
  reply is **503 `file busy`** naming who holds the file, and **nothing was run** — retrying is safe,
  unlike after a 504.
- A caller that hangs up while queued is dropped from the queue; its script never runs.
- **Speed.** A round trip is ~2 ms over HTTP and ~100 ms through the CLI (Python start-up) — for
  dozens of small calls from code, POST to `/exec` directly. Return what you need, not whole trees:
  a 50k-item result costs ~250 ms, a 5 MB one about the same.
- **Don't pause with `setTimeout` in scripts.** Figma throttles timers in background tabs:
  `setTimeout(30)` takes ~30ms in the visible tab and up to ~1s in a background one, and every
  agent queued behind you pays for it. Figma API calls stay fast in the background.

Why the lock is needed: each file's plugin sandbox is a single-threaded async message handler over one
shared document and one shared undo stack. Without it, two scripts yield to each other at **every

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysosadchyi/figmosha2](https://github.com/denysosadchyi/figmosha2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
