---
trigger: always_on
description: ╔══════════════════════════════════════════════════════════════╗
---

╔══════════════════════════════════════════════════════════════╗
║         ANALYTICS SYSTEM — ENGINEERING RULES (TARMEEZ)      ║
╚══════════════════════════════════════════════════════════════╝

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SYSTEM CONTEXT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Platform:   Tarmeez — multi-tenant SaaS store builder
Backend:    NestJS + Prisma + PostgreSQL + TimescaleDB
Frontend:   Next.js 16+ | React 19+ | Tailwind CSS v4
Charts:     shadcn/ui Charts (built on Recharts)
State:      RTK Query (polling every 60 seconds)
Styling:    STYLE-RULES 1-10 (semantic tokens only)
Direction:  RTL Arabic (dir="rtl")

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 1] DATA ISOLATION — NEVER MIX WITH STORE DATA
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Analytics data is completely separate from store data.

NEVER:
✗ Add analytics columns to existing models (Order, Product)
✗ Query analytics from store-related services
✗ Mix analytics logic inside MerchantModule or StoresModule

ALWAYS:
✅ All analytics in dedicated AnalyticsModule
✅ Separate Prisma models for analytics tables
✅ Separate NestJS service: AnalyticsService
✅ Separate aggregation service: AggregationService
✅ Separate RTK Query API: analyticsApi.ts

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 2] PRIVACY — ANONYMOUS DATA ONLY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
No personally identifiable information (PII) is stored.

NEVER store:
✗ IP address (full or partial)
✗ User name, email, or any identifier
✗ Browser fingerprint
✗ Any data that links a visit to a specific person

ALWAYS store:
✅ sessionId — anonymous UUID generated client-side
   stored in sessionStorage only (cleared on tab close)
✅ Country and city derived from IP at request time
   then IP is discarded immediately — never persisted
✅ Device type, browser name (no version details)
✅ Timestamps, page paths, referrer domain only
   (not full referrer URL with query params)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 3] COLLECTION ENDPOINT — PUBLIC + FAST
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POST /api/analytics/collect is the single ingestion point.

Rules:
✅ Public endpoint — no authentication required
✅ Must return 200 in < 50ms (fire and forget)
✅ Use queue/buffer — never write to DB synchronously
   on every request (would kill DB under load)
✅ Validate storeId exists before accepting data
✅ Rate limit: max 100 requests/minute per sessionId
✅ Payload max size: 2KB
✅ Use navigator.sendBeacon on client — never fetch()
   (sendBeacon doesn't block page unload)

Queue pattern:
  Request → validate → push to in-memory buffer
  → flush to DB every 10 seconds in batch

NEVER:
✗ await prisma.create() inside collect endpoint
✗ Any heavy computation in collect endpoint
✗ Logging full request body (contains user behavior)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 4] TRACKING SCRIPT — LIGHTWEIGHT + ISOLATED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
The tracking script runs in every storefront.

MANDATORY constraints:
✅ Max size: 5KB minified and gzipped
✅ No external dependencies — vanilla JS only
✅ Must not block page rendering (async/defer)
✅ Must not use cookies
✅ Must not use localStorage (use sessionStorage only)
✅ Must throttle mousemove: max 1 event per 100ms
✅ Must throttle scroll: max 1 event per 500ms
✅ Must use sendBeacon — never fetch() or XMLHttpRequest
✅ Must handle errors silently — never throw to console
✅ Injected via Next.js Script component with
   strategy="afterInteractive"

Script receives via data attributes:
  data-store-id="uuid"
  data-endpoint="/api/analytics/collect"

NEVER:
✗ Import React or any npm package
✗ Modify DOM in any way
✗ Block main thread
✗ Send on every keystroke or mouse pixel

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[ANALYTICS-RULE 5] DATABASE — TIMESCALEDB PATTERNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Raw event tables are TimescaleDB hypertables.
Aggregated tables are regular PostgreSQL tables.

Hypertables (raw data — high write volume):
  page_views    — partition by: time (1 day chunks)
  events        — partition by: time (1 day chunks)
  heatmap_data  — partition by: time (1 day chunks)

Regular tables (pre-computed aggregates):
  analytics_hourly  — computed every hour by cron
  analytics_daily   — computed every midnight by cron

Rules:
✅ NEVER query raw hypertables for dashboard display
   Always query aggregated tables for performance
✅ Raw tables are write-only from the API perspective
✅ Aggregation runs as NestJS @Cron job every hour
✅ Add storeId index on ALL analytics tables
✅ Add retention policy: raw data kept 90 days only
   aggregated data kept indefinitely
✅ All time columns use UTC — convert in frontend

NEVER:
✗ SELECT * FROM page_views (full table scan)
✗ Complex JOINs on hypertables in real-time
✗ Store analytics in same Prisma schema file
   as store models — use separate schema or
   clearly marked section with comment separator

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohamed-elbarrah/Tarmeez-app](https://github.com/mohamed-elbarrah/Tarmeez-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
