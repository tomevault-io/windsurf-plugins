---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal **US-equities trading journal**, not a software product. The repo is three things:

1. **A durable record** — dated markdown under `journal/` and per-name notes under `stocks/`, plus chart data JSON under `journal/charts/data/`. These files are the _only_ persistence layer (no database).
2. **A toolchain** — custom Claude Code skills under `.claude/skills/` that pull market data and orchestrate analysis workflows. "Running" this repo means invoking a skill or one of its Python scripts, then writing the synthesis back into a journal/stock file.
3. **A chart web app** — a pnpm workspace rooted at the repo root: shared libraries live under `packages/` and hosts under `apps/`. The kernel lives in `packages/core` (`@kansoku/core`); `apps/server` is a thin HTTP host (Tsuki (Hono + NestJS-style modules/DI) controllers + WS) that wraps the kernel, hosted as a single process by `main.node.ts` in production; `apps/desktop` is an Electron shell that embeds the same kernel and reaches it over typed IPC (`electron-ipc-decorator`) instead of HTTP; `apps/web` is Vite + React and picks HTTP or IPC transport by environment — `pnpm dev` runs web+server (Vite dev server proxies to the server process, neither needs a separate build step), `pnpm dev:desktop` runs web+desktop with no server process at all; charts render locally at `http://localhost:5199`. Cross-package types sit in `packages/shared`. **Open-core split (2026-07-17)**: `apps/pro/` — a gitignored slot directory holding the private repo `Innei/kansoku-pro` (`@kansoku/pro`), loaded at boot via `packages/core/src/pro/loader.ts` — now provides only the paid surface (个股自动跟踪、深度研究、研究库 AI) plus license, while the free AI (自带 key 的复评、对话、AI 设置、macro 过滤、研究库浏览) has moved into open core and runs without `apps/pro`; `packages/pro-api` stays the public types-only contract. Without `apps/pro` the build is the complete free version (charts/realtime/journal + free AI all work, only the paid routes 404 and their UI hidden); `GET /api/capabilities` reports `{ pro, licensed }` unchanged. Paid-AI work therefore usually means editing `apps/pro` (its own git repo — commit there separately); free-AI work lives in `packages/core`. The server/kernel calls the longbridge CLI itself and computes every indicator in TS; charts are created via `POST /api/charts` (see `.claude/skills/chart/SKILL.md`). Realtime layer: a single WS connection (`/api/ws`) pushes live quotes (watchlist ∪ positions, pre/post/overnight aware) and 60s chart rebuilds while a page is open — persisted chart JSON stays frozen at analysis time. **Default way to launch the app is Electron: `pnpm dev:desktop` (or `pnpm dev:desktop:unlocked` for licensed-Pro behavior) — do not start the server process (`pnpm dev` / `apps/server`) unless the task specifically needs the HTTP host.** Before launching, make sure the pro overlays are projected (`pnpm overlay:sync`; a missing projection silently boots the free composition). `pnpm start` is the production form and requires `pnpm --filter @kansoku/web build` first. Tests with `pnpm test`.

**Documentation language — write every document in this repo in 中文白话 (modern vernacular Chinese).** This covers journal entries, stock notes, specs, READMEs, and this file. Keep English only for tickers, API/CLI identifiers, and terms with no natural translation. This **overrides** the global "products committed to git are written in English" default (`~/.claude/CLAUDE.md`) — for this repo, written docs are 中文白话, not English and not 文言.

**对话回复也用 中文白话，不用文言。** This project overrides the global 文言 chat-reply rule (`~/.claude/CLAUDE.md`). Every reply to the user — explanations, status updates, end-of-turn summaries — is plain modern Chinese.

**少用专业术语和英文行话** —— 细则与正反例见下方导入的纪律文件（TD-LANG-02）。

**持仓相关不要问用户，直接查长桥**（TD-BROKER-01）。

**市场范围跟随配置，默认 US**（TD-LANG-03；个人配置在 `journal/personal.md`）。

## Architecture — three layers

### Layer 1 — data sources (raw retrieval)

| Source                                                               | Access                    | Covers                                                                                                                                                            |
| -------------------------------------------------------------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Longbridge** plugin (`longbridge ...` CLI / `longbridge-*` skills) | brokerage account         | real-time quotes, K-line/OHLCV, fundamentals, capital flow, technicals, market temperature, news                                                                  |
| **`fred`** skill                                                     | free API key              | US/global macro time series (CPI, GDP, Fed funds, yields, M2, DXY)                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kansoku-trade/kansoku](https://github.com/kansoku-trade/kansoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
