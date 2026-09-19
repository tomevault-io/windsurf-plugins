---
trigger: always_on
description: > Companion to `llms.txt` (which is the doc map). This file is how to *work*: build, test, deploy, commit. Read `CLAUDE.md` first — it carries the user's irrevocable rules.
---

# AGENTS.md — Working in this Repo as an AI Agent

> Companion to `llms.txt` (which is the doc map). This file is how to *work*: build, test, deploy, commit. Read `CLAUDE.md` first — it carries the user's irrevocable rules.

## TL;DR

- Always confirm before destructive ops (commit, terraform apply, EC2 recreate).
- TDD where the logic is testable; smoke-test where the network is the test.
- Containers run on a single EC2; deploy = `git pull && docker compose up -d --build` over SSM.
- memex's brain index is rebuildable from source content; if RDS is wiped, re-sweep restores it (~5-10 min, $0 — Titan is credit-eligible).
- memex is reached over MCP only (`POST /mcp`, through cloudflared or — with `ingress_mode = "caddy"` — a Caddy sidecar on the instance's own IP). No chat surface, no bot — just MCP clients (Claude Code, Cursor, …).

## Required workflow — run the skill for every change

No change is "done" until the skills have run. For **every** change —
features, fixes, refactors, docs, infra — in order:

1. **Self-review skill/agent.** Dispatch the review agent whose
   specialty matches what you changed (the table in `CLAUDE.md` →
   "Self-review after each implementation"): `security-engineer` for
   auth / secrets / ingress, `code-reviewer` for logic / refactor,
   `quality-guard` for new tests, `devops-automator` for CI / docker /
   terraform, `ai-engineer` for engine / MCP / retrieval,
   `reality-checker` for "it's live now" claims, `bug-hunter` for
   adversarial sweeps, `technical-writer` for docs. Act on every
   CRITICAL / HIGH finding before declaring done.
2. **Ship workflow.** Follow `CLAUDE.md` → "Ship workflow":
   **test → push → deploy → verify → release**, in that order, every
   time. A change is not shipped until the live EC2 is running it and
   `/health` + the MCP smoke-test pass; user-facing version bumps
   end with a SemVer tag + GitHub release (see "Release" below).

Both are non-negotiable and apply even to one-line fixes — the cost of
one extra skill/agent run is cheaper than a production regression.

## Build & test (memex)

```bash
cd deploy/memex
bun install               # frozenLockfile=true; never commit lock drift
bun run test:sharded      # the full suite — ~20 min, 324 files
bun test tests/foo.test.ts  # one file while iterating — seconds
bun run src/cli.ts --help # CLI surface
```

**Never run the whole suite as a bare `bun test`.** Every PGLite instance a
test opens reserves WASM linear memory that is never returned, so one
process running all 324 files dies inside `pg_initdb` with `RangeError: Out
of memory` and reports hundreds of phantom failures that have nothing to do
with your change. `test:sharded` runs fixed-size chunks in fresh processes —
the same script CI runs. A bare `bun test <file>` on one or a few files is
fine and fast.

There is no `bun run build` step for runtime — the daemon starts via `bun run src/cli.ts serve`. The `build` script in `package.json` exists for diagnostic bundling, not deployment.

## CLI commands worth knowing

`bun run src/cli.ts <cmd>` (aka `memex <cmd>` in the container). `--help` lists them all; the ones you'll reach for most:

```
export [--dir DIR] [--source ID]     # dump every live page to a markdown tree (frontmatter + body,
                                      #   slug dirs); --source scopes to one tenant. Backup / portability
                                      #   escape hatch for the DB-only substrate.
eval-probe [--limit N] [--max-usd N] # replay the eval set, append a row to eval_snapshots (nightly
                                      #   probe); --max-usd caps per-run spend (converts to a query cap).
cycle [--phases a,b,c] [--stale-days N]  # run one maintenance cycle on demand. Now takes the daemon's
                                      #   `memex-cycle` advisory lock — a one-shot skips (with a message)
                                      #   when the periodic loop holds it, so the two can't double-spend.
```

The take-review lifecycle is exposed over MCP, not the CLI: `list_takes` / `takes_search` (trigram search over take claims) to find takes, `set_take_status` to flip one to `accepted` / `rejected`.

## Build & test (full stack — Docker)

The simplest "does my change build" check uses Docker locally (matches the EC2 architecture):

```bash
cd deploy
docker compose build memex            # ~30s on warm cache
```

Full local up requires the secrets — they're gitignored and only fetched on the EC2. Don't try to bring up the stack on your laptop; smoke-test on EC2.

## Deploy

Always: `git push origin main` → SSH/SSM into EC2 → `cd /opt/<project> && git pull && docker compose --env-file .env -f deploy/docker-compose.yml up -d --build` → wait for `memex` to report `Up <N> (healthy)` → smoke-test the MCP surface from inside the network:

```bash
docker exec deploy-memex-1 sh -c '
  echo "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"tools/call\",\"params\":{\"name\":\"stats\"}}" \
    | wget -qO- --post-file=/dev/stdin --header=Content-Type:application/json http://127.0.0.1:18790/mcp
'
```

Then confirm `brain.<domain>/mcp` answers an MCP client (Claude Code).

Never:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timurgaleev/memex](https://github.com/timurgaleev/memex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
