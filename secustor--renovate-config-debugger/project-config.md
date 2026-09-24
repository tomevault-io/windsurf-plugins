---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# Agents.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Compiler explorer for Renovate configs": a static React SPA that runs **Renovate's own config code in the browser** — parsing, migration, massaging, validation, preset resolution, merging, and a packageRules simulator — and renders the trace. Big-picture rationale: `docs/Architecture.md`. Per-feature design decisions: `roadmap/` (numbered docs, one per feature).

## Commands

```bash
mise install && pnpm install   # node + pnpm versions come from mise.toml
pnpm dev                       # app dev server (vite)
pnpm test                      # all workspace tests (engine golden+shimmed, app unit+components+shimmed, cli, oauth-worker)
pnpm typecheck                 # tsc across all packages, plus tools/ (the agent hooks)
pnpm lint                      # oxlint --type-aware + stylelint (zero tolerance: any report fails CI)
pnpm format                    # oxfmt (format:check to verify)
pnpm build                     # all packages
```

Targeted tests:

```bash
pnpm --filter @renovate-config-debugger/engine test:golden    # real renovate modules, reference snapshots
pnpm --filter @renovate-config-debugger/engine test:shimmed   # browser module graph, must match golden
pnpm --filter @renovate-config-debugger/app test:unit         # all three app vitest projects (unit + components + shimmed)
pnpm --filter @renovate-config-debugger/app test:e2e          # playwright; requires `pnpm --filter …/app build` first
pnpm --filter @renovate-config-debugger/app exec vitest run --project unit src/lib/share.test.ts   # single file
pnpm --filter @renovate-config-debugger/app exec playwright test e2e/04-simulator.spec.ts          # single e2e
pnpm --filter @renovate-config-debugger/app check:dev-graph   # guards `vite dev` module graph against Node-only leaks
```

## Debugging config resolution: use `rcd`

**Do not** write a throwaway `*.shimmed.test.ts` to poke the engine, and do not
drive the app in a browser, to answer a question about a config. `packages/cli`
(roadmap 058, experimental) hosts the same shimmed module graph under Node and
answers those questions directly:

```bash
alias rcd='pnpm --filter @renovate-config-debugger/cli rcd'
rcd digest renovate.json          # the whole run in one paragraph — start here
rcd validate renovate.json        # exit 2 = Renovate would refuse it
rcd tree renovate.json --node "config:best-practices" --body resolved
rcd provenance renovate.json labels
rcd simulate renovate.json --dep '{"depName":"react","currentValue":"17.0.0"}'
rcd compare before.json after.json --dep '{"depName":"react"}'   # the edit oracle
rcd group renovate.json --dep '{"depName":"a"}' --dep '{"depName":"b"}'  # would these updates group, and does the group form?
rcd docs minimumReleaseAge        # option semantics for the pinned Renovate
```

`--format json` on any subcommand for machine-readable output. Preset-node
bodies are large — query one node at a time. `packages/cli/README.md` has the
full surface, the credentials table and the endpoint guard.

For an interactive session, `rcd mcp` is the same answers as typed MCP tools
with a warm engine (roadmap 060) — `run_config` returns a `runId` and the
drill-down tools query that held run, so the whole session describes one
resolution instead of re-resolving per question:

```bash
claude mcp add rcd -- npx -y @renovate-config-debugger/cli mcp
```

(In Claude Code this checkout already registers the server: the repo root's
`.mcp.json` is the plugin's server config, and the project scope picks it up.)

**In this checkout that server answers from the LAST PUBLISHED release**, not
from your working tree: `.mcp.json` resolves `rcd` through `npx`, and it has to
— it doubles as the published plugin's server config
(`.claude-plugin/plugin.json` names it), so it cannot point at a path inside
the repo. While you are changing `packages/cli/src` (or anything the CLI reads
— engine, or the app's `headless` derivations), the MCP answers describe the
released bundle instead of your edit. Two ways out:

```bash
# per developer: a local-scope entry that shadows .mcp.json's `rcd` for THIS
# project only (stored in ~/.claude.json under this project, never committed)
claude mcp add -s local rcd -- node packages/cli/bin/rcd-dev.mjs mcp
claude mcp remove -s local rcd   # back to the published bundle

# or per question, no config at all:
pnpm --filter @renovate-config-debugger/cli rcd digest renovate.json
```

MCP servers cannot be declared in `.claude/settings.json` or
`settings.local.json` — Claude Code reads server definitions only from
`.mcp.json` (project scope) and `~/.claude.json` (local/user scope) — which is
why the override is a command you run once rather than a file in the repo.
`bin/rcd-dev.mjs` is the same graph, served from `src/`, so an edit is live on
the next call with no build step.

The workflow those tools want — validate first, digest for orientation, drill
down one node at a time, `compare` as the oracle before proposing an edit — is
written down once, in `skills/debug-renovate-config/SKILL.md` (roadmap 061).
Read it when you are debugging a config here; it is the same skill the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secustor/renovate-config-debugger](https://github.com/secustor/renovate-config-debugger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
