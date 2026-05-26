---
trigger: always_on
description: This file is the **first thing an implementation agent should read** when
---

# Agents — rules of engagement

This file is the **first thing an implementation agent should read** when
opening this repo. It explains how multiple agents share work without
stepping on each other, and how a single agent can stop mid-task and have a
later session resume cleanly.

## TL;DR

```bash
# 1. Orient
cat AGENTS.md                            # this file
less browser-plan/13_MILESTONES.md       # where we are
less tasks/INDEX.md                      # what's available

# 2. Claim something unblocked
./tasks/lib/claim.sh claim wp:M1-03-dom-core "agent-claude-<your-handle>"

# 3. Read the package file and start working on main
less tasks/M1/wp-M1-03-dom-core.md

# 4. Commit often, with the wp id in the subject:
#    "wp:M1-03 — Node hierarchy + tests"

# 5. Build + test before completing
dotnet build && dotnet test

# 6. Mark complete:
./tasks/lib/claim.sh complete wp:M1-03-dom-core
```

**All work happens on `main`.** This repo doesn't use a per-package branch
workflow — agents commit directly to main. (If a remote with PR review is
ever wired up, the optional `in_review` state in `tasks/SCHEMA.md` is
available for it; today it's unused.)

If you have to stop early, **leave a handoff log entry** in the task file
and either keep the claim (you'll resume) or release it
(`./tasks/lib/claim.sh release wp:…`). Either way: commit so the next
agent sees the state.

## The contract

| You can rely on | You must do |
|---|---|
| One file per work package under `tasks/M*/wp-*.md` | Touch only your claimed package's file (plus your code changes) |
| `tasks/INDEX.md` reflects the current status | Update INDEX.md when you change a status |
| Work happens on `main`; commits carry the wp id in the subject | Prefix commit messages with `wp:<id> —` so history is greppable |
| Dependencies are explicit in `depends_on` | Don't start a package whose deps aren't `complete` |
| Stale claims age out at 72 h | Add a handoff log entry every session, even if you're "still going" |

## Repo map

```
starling/
├── AGENTS.md                  ← you are here
├── README.md                  ← human-facing intro
├── browser-plan/              ← the design (immutable except by deliberate edit)
│   ├── 00_INDEX.md            ← start here for design context
│   ├── 13_MILESTONES.md       ← what milestone we're in
│   └── 14_AGENT_TASKS.md      ← authoritative package catalog
├── tasks/                     ← work coordination (this repo's queue)
│   ├── README.md              ← detailed workflow
│   ├── SCHEMA.md              ← frontmatter contract
│   ├── INDEX.md               ← current status of all packages
│   ├── lib/claim.sh           ← atomic claim/release helper
│   └── M<n>/wp-*.md           ← one file per work package
├── src/                       ← engine + Headless CLI + Avalonia Gui (win/mac/linux)
│   ├── Starling.AppHost/          ← Aspire AppHost (orchestrates Gui + Headless)
│   └── Starling.ServiceDefaults/  ← Aspire OTel + health-check shared bootstrap
├── tests/                     ← one xUnit project per src/ module + E2E
├── bench/Starling.Bench/      ← BenchmarkDotNet
└── testdata/                  ← fixtures + golden PNGs + WPT subsets
```

## Build + test (must be green before merge)

```bash
dotnet --version            # expect 10.0.x
dotnet restore
dotnet build -c Debug
dotnet test  -c Debug
```

If `dotnet build` errors with permission-denied apphost deletions in a
sandbox or container, pass `-p:UseAppHost=false`. This is a sandbox quirk
only — CI runs without the flag.

## Getting traces & telemetry from Aspire

When you need to debug runtime behavior — a slow request, a failing fetch, an
unexpected span — pull traces from the Aspire dashboard's telemetry API instead
of adding `Console.WriteLine`. **This only works while the AppHost is running**
(`aspire run` against `src/Starling.AppHost`, or via the `aspire` skill); the
telemetry API is served by that running dashboard.

There are three ways in, listed in the order you should reach for them.

**1. MCP tools (preferred — already wired in `.mcp.json` via `aspire agent mcp`).**
No setup; just call them. The drill-down path:

- `mcp__aspire__list_traces` — list distributed traces (trace IDs, the resources
  each spans, duration, error flag). Optional `resourceName` narrows to one
  resource; omit it for all. Use this first to find the trace ID you care about.
- `mcp__aspire__list_trace_structured_logs` — given a `traceId`, the structured
  logs for that trace, grouped by span. **Prefer this over per-resource logs
  when investigating a specific trace.**
- Supporting tools: `mcp__aspire__list_structured_logs`,
  `mcp__aspire__list_console_logs`, `mcp__aspire__list_resources`.

Typical loop: `list_traces` (optionally filtered to a resource) → pick the
`traceId` → `list_trace_structured_logs` for span-level detail.

**2. Aspire CLI (`aspire otel`).** Same telemetry API, handy for ad-hoc shell
use and for full OTLP JSON you can pipe to `jq`:

```bash
aspire otel traces [<resource>] --format Json   # list traces (Table is default)
aspire otel traces -t <trace-id> --format Json   # one trace's spans, full detail
aspire otel traces --has-error -n 20             # only failing traces, cap 20

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [starling-browser/starling](https://github.com/starling-browser/starling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
