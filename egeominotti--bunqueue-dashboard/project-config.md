---
trigger: always_on
description: A web dashboard that **fully drives** a bunqueue server: view + control queues,
---

# bunqueue-dashboard — project instructions

A web dashboard that **fully drives** a bunqueue server: view + control queues,
jobs, DLQ, cron, webhooks, workers, live activity, and the server **process
lifecycle** (start / stop / restart).

It talks only to bunqueue's public HTTP API (`:6790`) plus a small local
**control agent** that manages the server process. It never imports or modifies
bunqueue source.

## Golden rule: additive only

**Never rewrite or break existing files.** Build new capabilities as **new files**
and connect them with minimal glue (a route in `src/App.tsx`, a nav item in
`src/components/layout/Sidebar.tsx`). Two API layers coexist on purpose:

- `src/lib/api.ts` — the original client used by the first-generation view pages.
- `src/lib/bq.ts` — the complete, shape-verified client used by every `pages/control/*`
  page and the agent. **New work uses `bq`.**

Do not "fix" old pages in place; add a corrected new page and route to it.

## Stack

React 19 · React Router 7 · Zustand 5 · Vite 8 · Tailwind CSS v4 · Biome · Bun · TypeScript.

## Layout

```
bunqueue-dashboard/
├── .github/
│   ├── workflows/             # ci.yml · pages.yml · docker.yml · release.yml (see “CI/CD” below)
│   ├── dependabot.yml         # weekly npm / actions / docker bumps
│   └── pull_request_template.md
├── agent/                     # Bun control agent (process lifecycle) — NOT linted/typechecked with src
│   ├── manager.ts             # ProcessManager: spawn/kill bunqueue, log ring buffer, dbStats()
│   ├── db.ts                  # read-only SQLite inspector: tables/schema/rows/query (readonly conn, unit-tested)
│   ├── server.ts              # fetch handler + origin/CORS/token policy (unit-tested)
│   └── index.ts               # binds 127.0.0.1; Origin allowlist + locked CORS + optional AGENT_TOKEN;
│                               # SIGINT/SIGTERM stop the managed server (no orphans)
├── docker/Caddyfile           # SPA history fallback + gzip/zstd + immutable asset caching for the image
├── scripts/dev.ts             # one-command dev launcher (`bun start`) — NOT linted/typechecked with src
├── Dockerfile                 # multi-stage: Bun build → Caddy serve
├── src/
│   ├── lib/                   # api.ts, bq.ts, bqTypes.ts, types.ts, jobActions.ts, sse.ts, format.ts, cn.ts,
│   │   │                       # usePolledData.ts, useActivityStream.ts, useThroughputSeries.ts, useAlertEngine.ts,
│   │   │                       # cloneJob.ts, cronPreview.ts, exportFile.ts, flowLayout.ts, promisePool.ts
│   │   ├── demo/              # demo mode (VITE_DEMO=1 / ?demo): installs a fetch+SSE shim answering every
│   │   │                       # bunqueue/agent route from fixtures.json — what the GitHub Pages build runs
│   │   └── copilot/           # in-app LLM assistant runtime: providers.ts (Anthropic/OpenAI/Google/custom),
│   │                           # tools.ts (zod-validated bq tools), runtime.ts (agent loop, max 8 steps)
│   ├── components/
│   │   ├── layout/            # Sidebar, Topbar, AppLayout, SidebarFooter
│   │   ├── ui/                # StatCard, StatusBadge, Card, Button, CopyButton, form, feedback, PageHeader, AreaChart, icons
│   │   ├── copilot/           # Copilot launcher + lazy CopilotPanel (~160 KB gz chunk, loaded on first open)
│   │   └── dashboard/stores/  # Zustand: theme, connection, alerts, s3, toast, copilot
│   ├── pages/                 # first-gen view pages (Overview, Queues, QueueDetail, Jobs, Dlq, Cron, Metrics, Workers,
│   │   │                       # Logs, Usage, S3Backup, Settings, NotFound) + Alerts (routed at /alerts, in nav)
│   │   ├── queue/              # QueueConfig (used by classic QueueDetail)
│   │   └── control/           # Pro pages: OverviewPro, ServerControl, AddJob, BulkAddJobs, JobInspector, JobsPro,
│   │       │                   # QueuesOverview, QueueDetailPro, DlqPro, DlqControl, MetricsPro, LogsPro,
│   │       │                   # QueueControl, CronManager, Webhooks, Diagnostics, S3BackupPro, Database
│   │       │                   # (SQLite inspector), UsagePro, WorkersPro, Benchmark, Flows (DAG viewer), McpServer
│   │       ├── benchmark/      # Benchmark subcomponents: engine, useBenchmark, RunHistory
│   │       ├── job/            # JobInspector subcomponents: JobTimeline, JobBackoff, JobActionsPanel, JobChildren,
│   │       │                   # JobDataEditor, JobLogs
│   │       ├── queue/          # QueueControl subcomponents: QueueActions, ConfigForms
│   │       └── server/        # ServerControl subcomponents
│   ├── App.tsx                # routes — see docs/pages.md for the verified route→page table
│   └── main.tsx                # entry (fonts, theme, router; basename = import.meta.env.BASE_URL for Pages)
├── test/                      # bun test (format, sse, manager, agent lifecycle, s3 store, bq client, stores)
│                               # coverage floor enforced by scripts/check-coverage.ts (`bun run test:coverage`)
└── docs/                      # how it works — see docs/README.md; docs/known-issues.md tracks verified gaps
```

## Run

```bash
bun install
bun start                   # agent + dashboard together (Ctrl-C stops both) — the simple path
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egeominotti/bunqueue-dashboard](https://github.com/egeominotti/bunqueue-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
