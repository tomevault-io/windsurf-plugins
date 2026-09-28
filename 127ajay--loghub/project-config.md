---
trigger: always_on
description: Guidance for Claude Code (or any future agent) working in this repository. Read this before
---

# CLAUDE.md

Guidance for Claude Code (or any future agent) working in this repository. Read this before
making changes - it explains what LogHub is, what's actually built vs. planned, and the decisions
already made so they don't get silently re-litigated or reverted.

## What this project is

LogHub is a centralized, web-based log viewer. Internal Web Applications on a server write their
own `*.log` files to disk; today, checking them means RDPing into the server and hunting through
folders. LogHub reads those files directly and shows them in a browser - live tail plus searchable
history - so developers and support staff never need server access just to read a log.

Full requirements live in `docs/prd/`, split by phase (see below). This file is the "how the
codebase actually works right now" companion to those requirement docs.

## Vision / where this is going

Phase 1 (pilot) is implemented: Web Applications only, single server, no auth, no database. See
`docs/prd/phase-1-pilot.md` for exact scope and `docs/prd/00-overview.md` for the architecture
decisions that shape everything (single-server deployment, no Elasticsearch/SQL, no login).

Phase 2 (`docs/prd/phase-2.md`) is **mostly implemented** as of 2026-07-20: search refinements
(regex + AND/OR/NOT), date-range export, index caching, and redaction. **Alerting was explicitly
deferred** by the user and has no design work - don't start it without scoping it first.

Phase 3 (`docs/prd/phase-3.md`) is **partially resolved as of 2026-07-21**:

- **Non-Web-App log types: resolved, no code was needed.** There is no application-type concept in
  the codebase - `LogApplicationConfig` is a name plus folder paths. Any application writing flat
  `*.log` files (Windows Service, WCF, Web API) works today by registering its folder. Verified
  against NLog, log4net, and bracketed ASP.NET layouts on a running instance. Applications logging
  **only to the Windows Event Log** are still unsupported and would need a real second collection
  mechanism. Don't describe the tool as "Web Applications only" - that was pilot scoping, not a
  code constraint.
- **Multi-server collector agent and authentication: deferred by the user (2026-07-21)**, with a
  full plan in `docs/plans/2026-07-21-phase-3-future-multiserver-and-auth.md`. Both are still gated
  on their trigger actually firing - don't build ahead of them. Note the plan's key finding: a UNC
  root path already works with no code change (`ResolveRoot` passes rooted paths through), so a
  collector agent should never be the first thing tried.

## Current implementation status (Phase 1)

Working, in `src/LogViewer.Web/`:

- Live Tail - polls each app's today's log file(s) every few seconds, pushes new lines to the
  browser over SignalR.
- History - date picker, keyword/level filter, reads matching file(s) directly on request.
- Admin page - register/remove applications (name + one or more log folder paths) from the
  browser. **This diverged from the original PRD draft**, which assumed `appsettings.json`
  editing; a user explicitly asked for frontend-managed registration instead. See "Key decisions"
  below.
- Handles flat log files (`Root\app-2026-07-20.log`), one-folder-per-day layouts
  (`Root\2026-07-20\*.log`), and split `Root\2026\07\20\` nesting. The scan is **recursive**
  (depth-capped at 6) - it used to look only one level deep, which meant any nested layout indexed
  to nothing and History showed "No log data found". Each file's date falls back through:
  date in file name -> date in folder path -> last-write time, so every readable file lands
  somewhere in the index rather than disappearing.
- Group-by-tag in History: the app offers grouping on whatever structured fields it finds in the
  selected day's logs (see `ExtractTags` below), and no grouping at all for formats that carry none.
- Multi-line log entry grouping - a line without a timestamp is treated as a continuation of the
  previous entry (stack traces, wrapped messages), not a separate broken entry. This was added
  after Serilog's own multi-line output format broke naive one-line-per-entry parsing; the fix is
  general-purpose, not Serilog-specific. See "How log parsing works" below.
- CSV export from the History view.
- Files page - read-only browsing of folders that hold `.txt`/`.json`/`.xml` documents rather
  than log streams. Separate registry, separate page, no tailing, no date index. See "How the
  file viewer works" below and `docs/plans/2026-07-27-file-viewer-design.md`.
- LogHub's own operational logging goes through Serilog to `Logs/loghub-*.log` (rolling daily) -
  separate from the application logs it monitors.
- No authentication. No database. No collector agents.

## Key decisions (don't relitigate these without a reason)

These came out of explicit stakeholder Q&A during the PRD process (`docs/prd/00-overview.md` has
the full decision log):

1. **Single server.** The web app and the log files it reads live on the same machine. No remote
   agents, no network log shipping.
2. **No database, no search engine.** Elasticsearch/OpenSearch/SQL were explicitly ruled out. The
   app reads `*.log` files directly from disk on every request - no persistence layer beyond the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [127Ajay/LogHub](https://github.com/127Ajay/LogHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
