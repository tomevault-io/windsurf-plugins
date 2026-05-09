---
trigger: always_on
description: Working context for Claude Code sessions in this repo. Read it all before making non-trivial changes. If something here is wrong, fix the doc and the code in the same PR.
---

# CLAUDE.md — Xarji

Working context for Claude Code sessions in this repo. Read it all before making non-trivial changes. If something here is wrong, fix the doc and the code in the same PR.

---

## 1. What we are building

**Xarji** (Georgian: ხარჯი, "expense") is a macOS desktop app that parses your Georgian bank SMS locally and shows the results on a dashboard you own end-to-end.

- Input: SMS notifications from Georgian banks (TBC, Bank of Georgia / Solo, Liberty, Credo, Basis, Tera) that land in `~/Library/Messages/chat.db`.
- Processing: per-bank regex parsers extract `{ amount, merchant, direction, cardLastDigits, date, … }` and classify transactions as `payment`, `payment_failed`, or `credit`.
- Storage: every parsed transaction is written to an **InstantDB** app the user creates themselves (free tier, their credentials). No Xarji-owned backend exists. A JSON backup at `~/.xarji/transactions.json` is the last-line-of-defense copy.
- UI: a React dashboard served by the app's own HTTP server on `127.0.0.1:8721`, accessed via the browser. A small Swift menu-bar app shows live status and gives the user a "Open dashboard" / "Quit" entry point.

**Intentional non-goals**:

- No cloud service Xarji runs. No Xarji account. No billing. No analytics.
- No cross-platform support. macOS 13+, Apple Silicon only. `chat.db` is a macOS thing; everything else follows.
- No collaboration / team features. Single-user tool for one Mac.
- No Mac App Store distribution. DMG only, signed + notarized with Developer ID.

**Target user**: a Georgian who banks with at least one of the supported banks, has an `appleid.com` account, and is comfortable pasting credentials once during onboarding. Not necessarily technical beyond that.

---

## 2. Architecture at a glance

Three components in one repo. They run as three separate processes in production:

```
┌─────────────────────────────────────────────────────────────────┐
│  Xarji.app (menu-bar, Swift)                                    │
│  ─ NSStatusItem in menu bar                                     │
│  ─ supervises xarji-core as a child Process                     │
│  ─ polls /api/health every 5s to show status                    │
└──────────────────┬──────────────────────────────────────────────┘
                   │ spawns + stdio pipe
┌──────────────────▼──────────────────────────────────────────────┐
│  xarji-core (Bun-compiled binary)                               │
│  ─ HTTP server on 127.0.0.1:8721 (never 0.0.0.0)                │
│  ─ serves embedded React bundle AND /api/* JSON                 │
│  ─ reads chat.db, parses SMS, writes to InstantDB               │
│  ─ state.db (SQLite) tracks per-sender cursor + dedup           │
└──────────────────┬──────────────────────────────────────────────┘
                   │ HTTP (localhost)
┌──────────────────▼──────────────────────────────────────────────┐
│  React dashboard (served from /)                                │
│  ─ InstantDB SDK in the browser, queries the user's app         │
│  ─ Onboarding wizard when the service reports "unconfigured"    │
│  ─ Full dashboard once the service is configured + running      │
└─────────────────────────────────────────────────────────────────┘
```

Key invariant: the **three processes never share secrets directly**. The InstantDB admin token lives in `~/.xarji/config.json` and is used by the Bun service. The client gets only a redacted form. The menu-bar app sees nothing sensitive.

---

## 3. Directory layout

```
.
├── CLAUDE.md                     ← this file
├── README.md                     ← end-user-facing intro
├── app-menubar/                  ← Swift menu-bar app (SwiftPM, no Xcode)
│   ├── Package.swift
│   ├── AppIcon.png               ← 1024×1024 source, regenerated to .icns at build
│   ├── version.env               ← MARKETING_VERSION + BUILD_NUMBER
│   ├── Scripts/
│   │   ├── package_app.sh        ← assembles Xarji.app from swift build output
│   │   └── xarji-core.entitlements  ← JIT entitlements for the Bun child
│   └── Sources/XarjiMenuBar/
│       ├── main.swift            ← NSApplication bootstrap + SIGTERM via DispatchSource
│       ├── AppDelegate.swift
│       ├── StatusBarController.swift   ← NSStatusItem
│       ├── CoreProcess.swift     ← Process supervision w/ exponential backoff
│       └── HealthPoller.swift    ← periodic /api/health
│
├── service/                      ← Bun service (xarji-core)
│   ├── package.json
│   └── src/
│       ├── index.ts              ← entrypoint: loads config, starts HTTP + service
│       ├── http.ts               ← Bun.serve — API + static assets + runtime HTML inject
│       ├── service.ts            ← orchestrator: watches chat.db, parses, syncs
│       ├── config.ts             ← loadConfig + isConfigured (file → env fallback)
│       ├── db-reader.ts          ← read-only bun:sqlite client for chat.db
│       ├── state-db.ts           ← ~/.xarji/state.db (cursor + dedup)
│       ├── parser.ts             ← parser registry + parseMessage() dispatch
│       ├── parsers/              ← per-bank regex parsers (solo.ts, tbc.ts, …)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tornikegomareli/Xarji](https://github.com/tornikegomareli/Xarji) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
