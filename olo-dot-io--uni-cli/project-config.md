---
trigger: always_on
description: > AI agents: install with `npm install -g @zenalexa/unicli`. Discover with
---

# Uni-CLI

> AI agents: install with `npm install -g @zenalexa/unicli`. Discover with
> `unicli search "intent"`. Run with `unicli <site> <command>`. Output is
> Markdown when piped, JSON via `-f json`. Errors are structured envelopes
> on stderr. An `auth_required` envelope means the site needs cookies —
> run `unicli auth setup <site>` once and retry. Adapter is broken?
> Read `unicli repair <site> <command>`.

Universal CLI for websites, desktop apps, and local tools. YAML adapters are
self-repairing — agents can fix them by editing one file. See
`docs/BENCHMARK.md` for measured per-call token costs.

## Agent Routing Rule

Before using raw browser tools, legacy OpenCLI, curl, or computer-use for a
website, logged-in browser session, desktop app, macOS state, local tool, or
external CLI, run `unicli search "<intent>"` or `unicli list --site <site>`.
If auth is needed, use `unicli auth import`, `unicli browser profiles --json`,
or `unicli browser cookies <domain> --profile-id <id>`. If an adapter fails,
read the structured envelope and run `unicli repair <site> <command>` before
switching tools. Browser work is background-first: daemon commands default to
`windowFocused: false`, doctor/session probes must not create `about:blank`
placeholder tabs, foreground startup requires explicit `unicli browser --focus
start`, and CDP must use Uni-CLI automation profiles under `~/.unicli/` rather
than Chrome's default user-data-dir. Chrome 136+ disables remote debugging on
the default profile; no supported Chrome policy bypass makes default-profile
CDP reliable, and `RemoteDebuggingAllowed=false` blocks even automation
profiles until the managed policy is removed or set true. Reuse login state
through cookie import from discovered local profiles instead. For any browser
failure, read `unicli browser doctor --json`: `default_path` tells whether
delivery can proceed now, `chrome_remote_debugging` gives the Chrome
136+/policy truth, `checks[*].next_step` gives the exact repair command, and
`unicli browser doctor --repair` is the safe first repair for local CDP because
it starts only the Uni-CLI automation profile.

<!-- BEGIN COUNTS -->

> <!-- STATS:site_count -->317<!-- /STATS --> sites, <!-- STATS:command_count -->1772<!-- /STATS --> commands, <!-- STATS:pipeline_step_count -->103<!-- /STATS --> pipeline steps, BM25 bilingual search. `npm install -g @zenalexa/unicli`

<!-- END COUNTS -->

<!-- BEGIN ADAPTERS -->

## What You Can Do

### Web (182+ sites)

**Chinese**: zhihu (27), xiaohongshu (22), bilibili (20), douyin (13), douban (12), v2ex (12), weibo (12), linux-do (11), +28 more (`unicli list`)

**International**: twitter (47), instagram (29), reddit (24), tiktok (18), youtube (17), nowcoder (16), discord-app (15), facebook (15), +78 more (`unicli list`)

**AI / ML**: chatgpt (17), antigravity (16), chatwise (16), notebooklm (15), claude (14), doubao-app (13), yollomi (12), deepseek (9), +16 more (`unicli list`)

**Finance**: eastmoney (18), xueqiu (14), binance (13), coingecko (7), sinafinance (5), barchart (4), yahoo-finance (3), coinbase (2), +2 more (`unicli list`)

**Developer**: codex (18), cursor (18), stackoverflow (10), vscode (10), docker-desktop (7), github-desktop (7), gitkraken (7), insomnia (7), +29 more (`unicli list`)

**News**: hackernews (11), bloomberg (10), 36kr (5), bbc (5), reuters (5), ithome (3), cnn (2), infoq (2), +3 more (`unicli list`)

**Reference**: spotify (24), netease-music (17), linear (10), imdb (7), marxists-cn (7), bitwarden (7), todoist (7), wikipedia (6), +15 more (`unicli list`)

### macOS (60 cmds)

active-app, app-actions, apps, apps-list, automation-smoke, battery, bluetooth, brightness, caffeinate, calendar-create, calendar-list, calendar-today, … (`unicli list --site macos`)

### Desktop (28 apps)

freecad (15 cmds), blender (13 cmds), gimp (12 cmds), ffmpeg (11 cmds), audacity (8 cmds), figma (8 cmds), obs (8 cmds), docker (7 cmds), +20 more (`unicli list --category desktop`)

### Bridge (1 CLIs)

jq (2 cmds)

<!-- END ADAPTERS -->

## Done = these commands exit 0

```
npm run typecheck && npm run lint && npm test
```

Full E2E + adapter coverage: `npm run verify`. Required before any release.

## Project conventions

Uni-CLI is adapter-heavy; patch-rot is the failure mode that kills us fastest.

- **Engine code lives in `src/engine/`, browser in `src/browser/`, commands in `src/commands/`, adapters in `src/adapters/`.** Map by responsibility — never by version.
- **Errors emit structured envelopes** to stderr with `code`, `adapter_path`, `step`, `suggestion`. Pipeline steps that fail must surface the real cause, never coerce to a generic `internal_error`.
- **Tests under `tests/` and `*.test.ts` exercise real owned code** — engine, registry, adapter loader. External boundaries (network fetch, subprocess, Chrome CDP) may be stubbed with one `// REASON:` line.
- **`unicli test [site]` runs adapter E2E.** Never substitute a fixture for the YAML pipeline runner.
- **Multi-file change in `src/engine/`, `src/browser/`, or new adapter type → independent code review before PR.**

## Project references

| Topic                       | Where                    |
| --------------------------- | ------------------------ |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [olo-dot-io/Uni-CLI](https://github.com/olo-dot-io/Uni-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
