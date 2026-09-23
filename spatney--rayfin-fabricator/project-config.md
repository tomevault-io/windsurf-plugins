---
trigger: always_on
description: > **You're an agent** working in a **lean, universal Rayfin app** (a Microsoft
---

# Universal App — agent guide

> **You're an agent** working in a **lean, universal Rayfin app** (a Microsoft
> Fabric Backend-as-a-Service app). It ships as a tiny "hello world" and is
> designed to **grow into whatever the user asks for** — a CRUD app, a charts
> dashboard, a Power BI analytics dashboard, or any mix. Your job is to grow it
> **on demand**, pulling in only the capabilities the request actually needs.

**Before you write any code:**

1. **Route first.** Read the **`capability-router` skill**
   (`.agents/skills/capability-router/SKILL.md`). It maps a plain-English request
   to a set of **capability packs**, and for each one tells you exactly what to
   turn on: which Fabric **service** to enable, which npm **modules** to install,
   which **code** to scaffold, and which **skill** to read for the patterns.
2. **Read before changing things.** Read the selected pack's skill before
   activating it. For Rayfin APIs, data, auth, configuration, or troubleshooting,
   follow the [rayfin-web-docs skill](.agents/skills/rayfin-web-docs/SKILL.md)
   before choosing APIs or changing services and dependencies.
3. **Stay lean.** Don't enable services, install modules, or copy in kit code the
   request doesn't need. The whole point of this template is that it starts small
   and only grows where the user is going.

---

## Rayfin documentation

Use the [official Rayfin documentation](https://rayfin.ai/) for platform
concepts, SDK APIs, permissions, limitations, and examples. This guide and the
pack skills cover **this template's integration and Fabricator workflow**, not a
second copy of the SDK reference.

The [rayfin-web-docs skill](.agents/skills/rayfin-web-docs/SKILL.md) makes this
lookup explicit for Rayfin-specific work: find the relevant pages, read their
guidance, and check version compatibility before implementation. Skip that
lookup for purely visual or unrelated React edits.

- Find relevant pages through the [documentation index](https://rayfin.ai/llms.txt)
  or the [documentation's agent entry point](https://rayfin.ai/AGENTS.md).
- Read the relevant [rules for coding agents](https://rayfin.ai/docs/reference/agent-rules.md)
  and [known limitations](https://rayfin.ai/docs/reference/known-limitations.md)
  before implementing Rayfin features.
- Append `.md` to a documentation page URL for its machine-readable version.
  Fetch only the pages needed for the selected capability, not the full corpus.

**Match the project's installed versions.** Read the CLI-generated
`.agents/skills/rayfin/SKILL.md` when present. Use the project's Rayfin MCP or
`npx rayfin docs` from the project root for version-matched APIs; the CLI works
without an MCP connection. Website pages carry `sdk_version` and `cli_version`
metadata: compare those with the resolved package versions before adopting
examples. Resolve conflicting guidance against the project's version-matched
sources rather than preserving a stale local example, guessing an API, or
upgrading packages just to match a website snippet.

Online docs are a reference, not a prerequisite for every edit: use the local
sources when offline. If no matching source is available, explain the gap
instead of inventing an API. Generic website deployment and development commands
do not override the [Fabricator workflow rules](#rules-fabricator-deploy-to-test).

## What ships in the base

A minimal React 19 + Vite app, Fabric-ready but deliberately bare:

- A no-auth `HomePage` (`src/pages/HomePage.tsx`) rendered by `src/App.tsx` — so
  it previews with **no backend** (`npm run preview`).
- **Fabric auth scaffolding, wired OFF** under `src/services/` +
  `src/hooks/AuthContext.tsx` — the base is a static, public page, so it needs no
  auth. **Wire auth for the default authenticated data workflow** (see Rules).
- **Graphein** wired in for charts: `src/components/Chart.tsx` (+ `useChart.ts`)
  renders a declarative `<Chart spec={…} />`. The `graphein-visuals` pack covers
  authoring specs.
- An **empty data schema** (`rayfin/data/schema.ts`) and `rayfin/rayfin.yml` with
  `auth` + `data` (mssql) + `staticHosting` enabled. The analytics stack is
  **absent** until the analytics pack turns it on.

## The capability packs

Each pack is a skill under `.agents/skills/`, trigger-gated so it only fires when
relevant. The router picks packs; the pack skill has the details.

| Pack (skill) | Turn it on when the user wants… |
|---|---|
| `authentication` | Sign-in, accounts, login, protected pages, per-user data |
| `data-modeling` | Records, CRUD, a database, entities, row-level security |
| `graphein-visuals` | Charts, graphs, KPIs, tables, a simple dashboard |
| `analytics` | Power BI / semantic-model dashboards, DAX, BI reporting |

The `analytics` pack also brings its own supporting skills (`build-workflow`,
`visuals`, `dax`, `fabric-data`, `app-design`, `headless-preview`) — read those
only when you're on the analytics path.

> **Fast path — one command.** A pack that ships a `pack.json` manifest turns on
> with a single idempotent command instead of dozens of manual steps:
>
> Read the pack skill before running its activation command.
>
> ```sh
> npm run pack:add -- <pack>      # e.g. analytics
> ```
>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spatney/rayfin-fabricator](https://github.com/spatney/rayfin-fabricator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
