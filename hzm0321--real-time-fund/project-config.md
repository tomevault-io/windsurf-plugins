---
trigger: always_on
description: **Generated:** 2026-03-21T03:22:46Z
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-03-21T03:22:46Z
**Commit:** 270bc3a
**Branch:** main

## OVERVIEW

Real-time mutual fund valuation tracker (基估宝). Next.js 16 App Router, pure JavaScript (JSX, no TypeScript), static export to GitHub Pages. Glassmorphism UI with heavy custom CSS variables (3557-line globals.css). All data via JSONP/script injection to external Chinese financial APIs (天天基金, 东方财富, 腾讯财经). localStorage as primary database; Supabase for optional cloud sync.

## STRUCTURE

```
real-time-fund/
├── app/                          # Next.js App Router root
│   ├── page.jsx                  # MONOLITHIC SPA entry (~3000+ lines) — ALL state + logic here
│   ├── layout.jsx                # Root layout (theme init, PWA, GA, Toaster)
│   ├── globals.css               # Tailwind v4 + glassmorphism CSS variables (~3557 lines)
│   ├── api/fund.js               # ALL external data fetching (~954 lines, JSONP + script injection)
│   ├── components/               # 47 app-specific UI components (modals, cards, tables, charts)
│   ├── lib/                      # Core utilities: supabase, get-query-client, query-keys, tradingCalendar, valuationTimeseries
│   ├── hooks/                    # Custom hooks: useBodyScrollLock, useFundFuzzyMatcher
│   └── assets/                   # Static images (GitHub SVG, donation QR codes)
├── components/ui/                # 15 shadcn/ui primitives (accordion, button, dialog, drawer, etc.)
├── lib/utils.js                  # cn() helper only (clsx + tailwind-merge)
├── public/                       # Static: allFund.json, PWA manifest, service worker, icon
├── doc/                          # Documentation: localStorage schema, Supabase SQL, dev group QR
├── .github/workflows/            # CI/CD: nextjs.yml (GitHub Pages), docker-ci.yml (Docker build)
├── .husky/                       # Pre-commit: lint-staged → ESLint
├── Dockerfile                    # Multi-stage: Node 22 build → Nginx Alpine serve
├── docker-compose.yml            # Docker Compose config
├── entrypoint.sh                 # Runtime env var placeholder replacement
├── nginx.conf                    # Nginx config (port 3000, SPA fallback)
├── next.config.js                # Static export, reactStrictMode, reactCompiler
├── jsconfig.json                 # Path aliases: @/* → ./*
├── eslint.config.mjs             # ESLint flat config: next/core-web-vitals
├── postcss.config.mjs            # Tailwind v4 PostCSS plugin
├── components.json               # shadcn/ui config (new-york, JSX, RSC)
└── package.json                  # Node >= 20.9.0, lint-staged, husky
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Fund valuation logic | `app/api/fund.js` | JSONP to 天天基金, script injection to 腾讯财经 |
| Main UI orchestration | `app/page.jsx` | Monolithic — all useState, business logic, rendering |
| Fund card display | `app/components/FundCard.jsx` | Individual fund card with holdings |
| Desktop table | `app/components/PcFundTable.jsx` | PC-specific table layout |
| Mobile table | `app/components/MobileFundTable.jsx` | Mobile-specific layout, swipe actions |
| Holding calculations | `app/page.jsx` (getHoldingProfit) | Profit/loss computation |
| Cloud sync | `app/lib/supabase.js` + page.jsx sync functions | Supabase auth + data sync |
| Trading/DCA | `app/components/TradeModal.jsx`, `DcaModal.jsx` | Buy/sell, dollar-cost averaging |
| Fund fuzzy search | `app/hooks/useFundFuzzyMatcher.js` | Fuse.js based name/code matching |
| OCR import | `app/page.jsx` (processFiles) | Tesseract.js + LLM parsing |
| Valuation intraday chart | `app/lib/valuationTimeseries.js` | localStorage time-series |
| Trading calendar | `app/lib/tradingCalendar.js` | Chinese holiday detection via CDN |
| Request caching | TanStack Query (`app/lib/get-query-client.js`, `app/lib/query-keys.js`) | Dedup + staleTime/gcTime |
| UI primitives | `components/ui/` | shadcn/ui — accordion, dialog, drawer, select, etc. |
| Global styles | `app/globals.css` | CSS variables, glassmorphism, responsive |
| CI/CD | `.github/workflows/nextjs.yml` | Build + deploy to GitHub Pages |
| Docker | `Dockerfile`, `docker-compose.yml` | Multi-stage build with runtime env injection |
| localStorage schema | `doc/localStorage 数据结构.md` | Full documentation of stored data shapes |
| Supabase schema | `doc/supabase.sql` | Database tables for cloud sync |

## CONVENTIONS

- **JavaScript only** — no TypeScript. `tsx: false` in shadcn config.
- **No src/ directory** — app/, components/, lib/ at root level.
- **Static export** — `output: 'export'` in next.config.js. No server-side runtime.
- **JSONP + script injection** — all external API calls bypass CORS via `<script>` tags, not fetch().
- **localStorage-first** — all user data stored locally; Supabase sync is optional/secondary.
- **Monolithic page.jsx** — entire app state and logic in one file (~3000+ lines). No state management library.
- **Dual responsive layouts** — `PcFundTable` and `MobileFundTable` switch at 640px breakpoint.
- **shadcn/ui conventions** — new-york style, CSS variables enabled, Lucide icons, path aliases (`@/components`, `@/lib/utils`).
- **Linting only** — ESLint + lint-staged on pre-commit. No Prettier, no auto-formatting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hzm0321/real-time-fund](https://github.com/hzm0321/real-time-fund) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
