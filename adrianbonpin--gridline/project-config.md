---
trigger: always_on
description: Guidance for AI coding agents working on **Gridline**.
---

# AGENTS.md

Guidance for AI coding agents working on **Gridline**.

---

## Project Identity

Gridline is an **open-source, cross-platform database GUI client** for PostgreSQL (with MySQL, SQLite, and Redis to follow). It is built as a **Tauri 2.0 desktop app** — a lightweight native shell (~40MB baseline) around a React web frontend, with a Rust backend handling all database operations, CLI tool orchestration, and local persistence.

**Core differentiators from commercial alternatives (DB Pro, TablePlus, Beekeeper Studio):**
- **Everything free, nothing paywalled** — where DB Pro caps free users at 2 connections / 3 tabs / 5 saved queries, TablePlus caps at 2 open tabs + 2 windows, and Beekeeper reserves backup/restore, file import, multi-table export, ERD, and several DB connectors (Oracle, MongoDB, ClickHouse…) for paid tiers, Gridline ships the full feature set with no limits on tabs, connections, or saved queries
- **DB-to-DB sync** — pipe-based `pg_dump` → `pg_restore` between two live connections; none of the alternatives (DB Pro, TablePlus, Beekeeper) offer direct DB-to-DB sync — they only back up to / restore from files
- **Deeper PostgreSQL object explorer** — full detail views for Functions, Triggers, Sequences, Enums, and Extensions; Beekeeper and TablePlus show tables/views/routines/triggers but no sequences, enums, or extensions (Beekeeper can't even display routine definitions — issue #329 open since 2020), while DB Pro's tree stops at tables, views, indexes, and enums

**Competitor reality check (verified 2026-05, from vendor docs/pricing/repos — keep this accurate):**
- **DB Pro** (dbpro.app): **Electron app** (founder-confirmed on HN; launched Nov 2025) — not native despite "native macOS, Windows, Linux apps" marketing copy. Free plan = 2 connections / 5 saved queries / 3 open tabs / 2 dashboards / 2 table tags; data imports + SSH tunneling are paid-only per the pricing table/FAQ, while CSV/JSON export **does work on the free tier** (paid plans advertise "unlimited exports"; FAQ inconsistently claims "unlimited local connections"). Has query folders, dashboard folders, and table tags (roadmap 100%). No backup/restore (no pg_dump/pg_restore anywhere) and no DB-to-DB sync — the "Deeper Database Management" roadmap (indexes, users, constraints, VACUUM/ANALYZE) is still 0%. Schema tree: tables, views, indexes, relationships, and enums (since v1.6.0); MSSQL also lists stored procedures — no functions, triggers, sequences, or extensions on PG. Timeline: v1.0 Nov 2025 → v1.4 MSSQL/SSH/Keychain/Neon (Jan 2026) → v1.6 Redis/enums (Feb 2026) → self-hosted Studio (Mar 2026). Marketing overclaims ("native", Neon listed before it shipped) and known bugs (strict TLS verification blocks some Supabase pooler connections).
- **Beekeeper Studio**: free Community edition = unlimited connections, no tab limits, saved queries, local folders (5.7+), staged Apply/Discard edits, basic query-result export. Paid-only: pg_dump/pg_restore backup/restore, file import, multi-table export, ERD, AI shell, JSON sidebar, cloud workspaces, and premium DB connectors (Oracle, MongoDB, ClickHouse, DuckDB…). Sidebar shows tables/views/matviews/routines/triggers — no sequences, enums, or extensions.
- **TablePlus**: free = 2 open tabs / 2 windows / 2 advanced filters, but every other feature is included (incl. pg_dump/mysqldump backup GUI). No DB-to-DB sync, no ERD, no folder hierarchy. Sidebar: tables, views, functions, procedures.

**Target audience:** Developers managing multiple database environments across projects (Personal, Work, Client). The workspace/folder hierarchy is a first-class concept.

---

## Tech Stack

| Layer | Technology | Notes |
| :--- | :--- | :--- |
| Desktop shell | Tauri 2.0 | Native webview wrapper, Rust backend |
| Frontend | React 19 + TypeScript 5.8 | Vite 7 for bundling/HMR |
| Styling | Tailwind CSS | Dark-first, glassmorphic aesthetic |
| State | Zustand or Jotai | Pick one and stay consistent per feature |
| Editor | Monaco Editor | SQL mode with custom autocomplete providers |
| Data grid | Glide Data Grid or TanStack Virtual | Virtualized, canvas-rendered |
| Backend | Rust (tokio async runtime) | Connection pools, IPC commands, shell execution |
| DB drivers | sqlx + tokio-postgres | Async, pure-Rust PostgreSQL driver |
| Local storage | SQLite via rusqlite | User settings, workspace state, query history |
| Credentials | OS keychain | macOS Keychain, Linux Secret Service, Windows Credential Manager |

---

## Project Structure

```
gridline/
├── package.json                  # Workspace root: bun workspaces ["desktop", "www"], orchestration scripts
├── desktop/                      # Tauri desktop app (React frontend + Rust backend)
│   ├── package.json              # Frontend deps + scripts (name: gridline-desktop)
│   ├── index.html
│   ├── vite.config.ts            # Vite + Tailwind v4 + Tauri dev server (port 1420)
│   ├── vitest.config.ts
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   ├── bunfig.toml               # bun test preload
│   ├── src/                      # React frontend (TypeScript)
│   │   ├── components/           # Reusable UI components
│   │   │   ├── layout/           # App shell, sidebar, tabs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianBonpin/gridline](https://github.com/AdrianBonpin/gridline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
