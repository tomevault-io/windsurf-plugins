---
trigger: always_on
description: > **Keep this file up to date.** When you discover new patterns, gotchas, or architectural changes during a session, update this file before finishing. If you see any styling inconsistency while working on a task, fix it immediately.
---

# AGENTS.md

> **Keep this file up to date.** When you discover new patterns, gotchas, or architectural changes during a session, update this file before finishing. If you see any styling inconsistency while working on a task, fix it immediately.

## Project Overview

Svelte 5 + SvelteKit crypto trading terminal ("OMBRA") for Solana/ETH/BASE/BSC tokens. Deployed to **Cloudflare Workers** via `@sveltejs/adapter-cloudflare`. Backend API proxied through Vite (dev) and `hooks.server.ts` (prod).

## Commands

| Task | Command |
|------|---------|
| Dev server | `pnpm dev` |
| Build | `pnpm build` |
| Type check | `pnpm check` |
| Deploy to Cloudflare | `pnpm deploy:cf` |
| Regenerate OpenAPI types | `npx openapi-typescript ../backend/opendex.v2.openapi.yaml -o src/lib/api/v2.d.ts` |

There is **no linter configured**. Lightweight component and utility tests run
with Vitest and Testing Library via `pnpm test`; CI uses `pnpm test:ci` to emit
JUnit evidence. Typecheck with `pnpm check` after every change.

## Tech Stack

- **Svelte 5** (runes: `$state`, `$derived`, `$effect`, `$props`, `$bindable`, `{#snippet}`)
- **SvelteKit** with Cloudflare Workers adapter
- **Tailwind CSS v4** via `@tailwindcss/vite` plugin (no `tailwind.config` file)
- **openapi-fetch** + **openapi-typescript** for typed API calls
- **lightweight-charts v5** for candlestick/volume charts
- **lucide-svelte** for icons, **simple-icons** for brand SVGs, `DexPaidIcon.svelte` for the dex-paid eagle
- **bs58** for Phantom wallet signature encoding
- **pnpm** as package manager

## File Structure

```
src/
├── app.css                    # Theme vars (@theme block + :root + [data-theme="light"]), global styles
├── app.html                   # HTML shell with blocking theme script
├── lib/
│   ├── api/
│   │   ├── v2.d.ts            # AUTO-GENERATED from ../backend/opendex.v2.openapi.yaml — DO NOT EDIT
│   │   ├── client.ts          # openapi-fetch client + auth/refresh middleware
│   │   └── types.ts           # Re-exports from client.ts
│   ├── components/
│   │   ├── index.ts           # Barrel export
│   │   ├── Navbar.svelte      # Top nav: search, wallet popover, theme builder trigger
│   │   ├── TokenList.svelte   # Sidebar compact token list
│   │   ├── TokenListItem.svelte
│   │   ├── TokenRow.svelte    # Scanner table row
│   │   ├── TokenTable.svelte  # Virtual-scroll scanner table
│   │   ├── TokenDetail.svelte # Token detail: chart, trades, holders, safety
│   │   ├── TokenStats.svelte  # Timeframe stats
│   │   ├── TokenChart.svelte  # TradingView chart with WS candle updates, area gradient
│   │   ├── MobileTokenCard.svelte  # Scanner mobile card with sparkline bg
│   │   ├── MemescopeCard.svelte
│   │   ├── TradePanel.svelte  # Buy/sell form
│   │   ├── PositionsPanel.svelte  # Positions, orders, history with swap lines
│   │   ├── WatchlistPanel.svelte  # Watchlist: callers, telegram, lists, wallets
│   │   ├── TwitterFeedPanel.svelte  # X feed widget below watchlist (collapse/hide via feSettings)
│   │   ├── CreateBotModal.svelte  # Bot creation/editing
│   │   ├── ThemeBuilderModal.svelte  # Theme customization modal
│   │   ├── DexPaidIcon.svelte # Reusable dex-paid eagle SVG icon
│   │   ├── TokenTabPreview.svelte # Hover mini-chart sparkline for token tabs (reads candleCache)
│   │   ├── FloatingTokenWindow.svelte # Popped-out token window (chart + header + mini buy/sell, drag/resize/z-order)
│   │   ├── MobileConnectModal.svelte  # Desktop QR for mobile connect
│   │   ├── MobileScanModal.svelte     # Phone QR scanner for mobile connect
│   │   ├── UserListModal.svelte
│   │   ├── SourcePicker.svelte
│   │   ├── ToastContainer.svelte
│   │   └── trader-analytics/     # Rankings, token trader drawer/chart, portfolio modal
│   ├── stores/
│   │   ├── auth.svelte.ts     # Wallet auth (challenge → signin), JWT refresh
│   │   ├── theme.svelte.ts    # Theme mode (dark/light/custom), custom seed, preview
│   │   ├── currency.svelte.ts # USD/native toggle
│   │   ├── feSettings.svelte.ts # FE-only settings (expandPositions, bubbleWatchlist + multiTab beta)
│   │   ├── tokenTabs.svelte.ts # Multi-tab token list + floating popout windows (z-order, focus, persistence)
│   │   ├── candleCache.svelte.ts # Shared close-price series + live mcap per token (TokenChart writes, hover preview reads)
│   │   ├── settings.svelte.ts # User settings + favourites (API-backed)
│   │   ├── trade.svelte.ts    # Trade state (+ quickBuy/quickSell for popouts, per-token loading)
│   │   ├── peg.svelte.ts      # SOL/ETH peg price via WebSocket
│   │   ├── toast.svelte.ts    # Toast notification queue
│   │   ├── tick.svelte.ts     # 1-second interval for live age display
│   │   ├── chart.svelte.ts    # Chart frame/marketcap toggle
│   │   └── traderAnalytics.svelte.ts # Shared trader drawer/modal targets
│   ├── ws/
│   │   └── client.ts          # WebSocket client (pub/sub, reconnect, auth)
│   └── utils/
│       ├── format.ts          # Number/price/time/address formatting, fmtVal/fmtPrice
│       ├── themeColors.ts     # HSL math, ramp generators, color theory, presets

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendex-ws/ombra](https://github.com/opendex-ws/ombra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
