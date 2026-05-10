---
trigger: always_on
description: ONE API for online reselling. The hosted service at `api.flipagent.dev`
---

# flipagent

ONE API for online reselling. The hosted service at `api.flipagent.dev`
gives AI agents and apps a unified surface for the full reseller cycle
(discovery → evaluation → buying → listing → fulfillment → finance)
across marketplaces. Today: eBay (REST mirror + scrape fallback). Soon:
Amazon, Mercari, Poshmark.

The whole API server is OSS (recall.ai-style: open backend, hosted
operations as the moat). Page rendering for `EBAY_*_SOURCE=scrape` is
delegated to a managed Web Scraper API (today: Oxylabs) — we POST a
URL and they return rendered HTML on their infrastructure, under their
upstream-marketplace ToS. flipagent's own code path is a normal HTTPS
client; it does not implement UA rotation, browser fingerprinting, or
any other vendor-side concern.

## Workspaces

| Path | Name | License | Role |
|---|---|---|---|
| `packages/types` | `@flipagent/types` | MIT | TypeBox schemas for flipagent's own `/v1/*` — `evaluate`, `discover`, `ship` (intelligence layer) plus errors, tier, billing, keys, takedown, health |
| `packages/types/ebay` | `@flipagent/types/ebay` | MIT | TypeBox schemas mirroring eBay REST shapes — `/buy` (Browse + Marketplace Insights) and `/sell` (Inventory, Fulfillment) subpaths |
| `packages/ebay-scraper` | `@flipagent/ebay-scraper` | MIT | eBay HTML parsers + plain-HTTP fetcher (BYO proxy) |
| `packages/sdk` | `@flipagent/sdk` | MIT | Typed client. Marketplace passthrough namespaces (`listings`, `sold`, `orders`, `inventory`, `fulfillment`, `finance`, `markets`) plus flipagent intelligence (`research`, `match`, `evaluate`, `discover`, `ship`, `draft`, `reprice`, `expenses`) and ops (`webhooks`, `capabilities`). |
| `packages/mcp` | `flipagent-mcp` | MIT | MCP server — exposes eBay tools + deal-finding tools to Claude Desktop / Cursor / Cline. |
| `packages/cli` | `flipagent-cli` | MIT | One-command MCP setup. Detects Claude Desktop / Cursor and writes the `flipagent` server entry. `npx -y flipagent-cli init --mcp --keys`. |
| `packages/api` | `@flipagent/api` | FSL-1.1-ALv2 (private — not published, source on GitHub; converts to Apache 2.0 two years after each release) | Hono backend: unified API surface (eBay-compat + `/v1/*`), scraping, scoring, auth, billing. |
| `apps/docs` | `@flipagent/docs` | proprietary (All Rights Reserved) | flipagent.dev marketing + dashboard site (Astro static). Source visible for transparency; redistribution / rebrand not permitted. |

## Dependency direction

```
   @flipagent/types ──┐
                      ├──►  @flipagent/sdk  ──►  flipagent-mcp  (npm)
                      │            │
                      │            │ HTTPS
                      │            ▼
                      │     api.flipagent.dev
                      │            │  (= @flipagent/api)
                      │            │
                      ├──►  @flipagent/api  ──►  Postgres
                      │     (Hono backend)       Oxylabs Web Scraper API
   @flipagent/ebay-scraper ─────►  │              eBay / Amazon / Mercari (future)
                                   │
                                   └──►  services/{scoring,quant,forwarder} (server-side math)
```

`flipagent-mcp` calls flipagent's hosted API through `@flipagent/sdk`.
Math (median, margin, scoring, recipes) runs **server-side** in
`packages/api/src/services/scoring/` so all SDK clients in any language
get the same scoring without re-implementing it.

## Structural rules

- **Marketplace-agnostic surface.** Endpoints live under `/v1/*` in
  three layers. The mirror layer mirrors eBay's REST path shape
  verbatim (`/v1/buy/*`, `/v1/sell/*`, `/v1/commerce/*`,
  `/v1/post-order/*`) so agents can read eBay docs and call our routes
  one-to-one.
  - Marketplace mirror — Buy:
    sourcing reads (`/v1/buy/browse/item_summary/search`,
    `/v1/buy/browse/item/{itemId}`,
    `/v1/buy/browse/item/get_items`,
    `/v1/buy/browse/item/get_items_by_item_group`),
    sold comparables (`/v1/buy/marketplace_insights/item_sales/search`),
    buy queue (`/v1/buy/order/*` — single surface, REST + bridge transports),
    bulk ops (`/v1/buy/feed/*`, `/v1/buy/deal/*`),
    buyer-side bidding (`/v1/buy/offer/*`).
  - Marketplace mirror — Sell:
    listing CRUD (`/v1/sell/inventory/*`),
    order ops (`/v1/sell/fulfillment/*`),
    payouts (`/v1/sell/finances/*`),
    selling policies (`/v1/sell/account/{fulfillment,payment,return}_policy`),
    seller marketing (`/v1/sell/marketing/*`),
    Best Offer outbound (`/v1/sell/negotiation/*`),
    seller perf (`/v1/sell/analytics/*`, `/v1/sell/compliance/*`,
    `/v1/sell/recommendation/*`),
    bulk ops (`/v1/sell/feed/*`),
    shipping labels (`/v1/sell/logistics/*`),
    eBay Stores (`/v1/sell/stores/*`),
    sell metadata (`/v1/sell/metadata/*`).
  - Marketplace mirror — Commerce (cross-cutting):
    taxonomy (`/v1/commerce/taxonomy/*`),
    catalog (`/v1/commerce/catalog/*`),
    connected user (`/v1/commerce/identity/*`),
    translation (`/v1/commerce/translation/*`).
  - Marketplace mirror — Post-order:
    returns/cases/cancellations/inquiries/issues (`/v1/post-order/*`).
  - Trading API XML wrappers exposed as JSON (no REST equivalent on
    eBay): `/v1/messages`, `/v1/best-offer` (inbound Best Offer
    respond), `/v1/feedback`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipagent/flipagent](https://github.com/flipagent/flipagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
