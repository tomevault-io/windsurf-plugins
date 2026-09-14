---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 产品边界（红线，2026-08-22 架构拆分后长期有效）

本仓库 = **mrd（Market Research Cockpit）产品代码**：只准行情数据服务 + 前端 + `/api/leads` + `server/hosting/`（托管版）。

**禁止**：把 OPC 透明办公室、公司官网、营销、客服的代码/数据/凭据/测试放进本仓。这些代码一律放各自仓库：

| 产品/服务 | 仓库 | 域名 |
|---|---|---|
| mrd（本仓） | theBigGavin/marketingdashboard | mrd.hermes.cc.cd |
| OPC 后台（opc-api） | theBigGavin/opc-os 的 `opc-api/`（私有） | opc.hermes.cc.cd |
| 官网后台 | theBigGavin/company-site-backend | api.hermes.cc.cd |
| company-site 前端 | theBigGavin/gavin-lab-company | www.hermes.cc.cd |
| knock 排行榜 | theBigGavin/mylauncher 的 `server/` | hermes.cc.cd/api/v1/knock |

**规则**：
1. 新增 API 默认挂对的产品域名（mrd→mrd.hermes.cc.cd、OPC→opc.hermes.cc.cd、官网→api.hermes.cc.cd）；跨域调用一律走对的产品后端，不在本仓加对方路由/反代。
2. 敏感凭据只存在于所属产品仓库的 .env（gitignore）。本仓 `server/.env` 只放 mrd 自己的 key（`IWENCAI_BASE_URL` / `IWENCAI_API_KEY` / `OPENROUTER_API_KEY` / `ARTIFICIAL_ANALYSIS_API_KEY`）。`BLOG_ADMIN_KEY` 归 company-site-backend，`WORKBENCH_TOKEN` 归 opc-os，不得出现在本仓。
3. 本仓保留的 mrd 本体（**勿误删/勿误报**）：`/api/rank` + `server/lib/qq-rank.cjs`（行情数据源）、`/api/leads` + `server/data/leads.json`（Pro 预注册）、`server/hosting/`（托管版，HOSTING=1 才加载）、`/company/opc/status.json` 静态服务（官网成员数数据源，红线保留）、`/api/v1/knock/*` 302 过渡重定向（旧客户端升级后删除）。
4. 防线：`scripts/check_product_boundary.sh` 扫描 index.cjs 路由关键字与 lib/sources/data 文件。提交前手动跑或接 CI，**FAIL 不得合入**。

## Commands

```bash
npm run dev      # Start dev: Vite (:3000) + data proxy (:3001), auto-proxied
npm run build    # TypeScript check + Vite production build → dist/
npm start        # Production: single Node process serves API + static files on :3000
npm run lint     # ESLint on all TS/TSX files
npm run preview  # Vite preview of dist/
```

### Release workflow

When the user asks to publish a release ("发版", "release", "publish"):

1. Determine the bump: `patch` (bug fixes), `minor` (new features), or `major` (breaking changes). Ask if unsure.
2. Run `npm version <bump>` — this increments `package.json`, commits, and creates the tag in one step.
3. Push: `git push origin main --follow-tags`
4. Create the GitHub release with `gh release create v<version> --generate-notes`. Pass `--notes` explicitly only when the auto-generated notes need custom content (e.g. Chinese release notes for this project).

## Architecture

**Frontend**: React 19 + Vite 7 + TypeScript + Tailwind CSS (shadcn/ui theme). Charts are hand-written SVG (no charting library). Path alias `@/` maps to `src/`.

**Backend**: `server/index.cjs` — a zero-dependency Node.js native HTTP server that aggregates public Chinese market-data endpoints (Tencent, Sina, Eastmoney, Wallstreetcn, CNBC, Binance, Sunsirs, etc.). No framework, no database. Uses in-memory TTL caches (1.5s–24h per endpoint), LRU eviction, and periodic sweeps. Some endpoints fall back from Node `fetch` to `curl` for TLS-fingerprint-sensitive upstreams. Production serves both API and built frontend from a single port.

**Routes** (React Router, `src/App.tsx`):
- `/` — Market cockpit (indices, sectors, money flow, news, industry chains, watchlist)
- `/ai` — AI cockpit (OpenRouter provider token consumption trends)
- `/goods` — Commodity prices (futures trends + spot/basis tables)
- `/fin` — Earnings window (disclosure calendar, forecasts, industry/stock profit rankings, company trends)

### Key patterns

**Unified quote hub** (`src/lib/market.ts`): All panel prices/changes come from a single client-side `MarketHub`. Components subscribe via `useQuote(code)` / `useQuotes(codes)`, which use reference-counting and a single 5s polling loop. The same ticker renders the same frame everywhere — no per-panel duplicate fetches.

**Shared modules** — when adding a new panel, reuse these instead of copying from existing panels:
- `src/lib/code.ts` — `normalizeStockCode(raw)` / `toMarketCode(raw)` for stock code prefix normalization (do NOT write another 6→sh / 0/2/3→sz mapper)
- `src/lib/storage.ts` — `loadJson<T>(key, fallback)` / `saveJson(key, value)` for localStorage persistence (do NOT hand-write try-catch JSON.parse/setItem)
- `src/lib/format.ts` — `TNUM` (tabular-nums style), `clsChg` (red/green color), `fmtYi`/`fmtWan`/`fmtYuan`/`fmtPct`/`fmtPrice` — all formatting + coloring in one place
- `src/hooks/useElementSize.ts` — `useElementSize(threshold?)` returns `{ ref, size }` for SVG auto-sizing (do NOT hand-write ResizeObserver + useState boilerplate)
- `src/hooks/useStockSearch.ts` — `useStockSearch()` returns debounced search state + keyboard navigation for stock picker inputs
- `src/components/dash/SharedUI.tsx` — `TabBar<T>` (segmented control, accent + size props, pill 高亮) and `AsyncContent` (loading skeleton / error+retry / empty / content wrapper) (do NOT copy-paste tab buttons or error templates)

**Panel layout** (`src/components/dash/DashboardLayout.tsx`): The cockpit is a grid of resizable `Panel` components arranged in rows. `DashboardLayout` reads `PanelRowDef[]` (row height ratios + per-panel width ratios) and renders panels wrapped in `React.memo` so zooming one panel doesn't re-render siblings. Panel zoom state is managed by `usePanelZoom` hook.

**Data hooks** — two polling patterns:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theBigGavin/marketingdashboard](https://github.com/theBigGavin/marketingdashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
