---
trigger: always_on
description: - **Fumadocs 16** (fumadocs-core 16.11.2, fumadocs-mdx 15.1.0, fumadocs-ui 16.11.2)
---

# Docs Site — Agent Instructions

## 1. Framework

- **Fumadocs 16** (fumadocs-core 16.11.2, fumadocs-mdx 15.1.0, fumadocs-ui 16.11.2)
- **Next.js 16.2.10** with App Router
- **pnpm 11.11.0** (package manager)
- **Tailwind CSS v4.3.2** (v4 `@theme` syntax, no `tailwind.config.js`)
- **Mermaid 11** for diagrams (client-side rendering via custom component)
- **DuckDB-WASM** for in-browser SQL playground
- **Observable Plot** for data visualization (shot charts, game flow, heatmaps, comparisons, trends, distributions)
- **Recharts** for admin dashboard charts
- **class-variance-authority (CVA)** for variant-driven UI components

## 2. Content Structure

```text
docs/
├── app/
│   ├── (home)/page.tsx              # Landing page (hero, scoreboard, topic grid, quick start)
│   ├── (admin)/admin/               # Admin route group (auth-gated)
│   │   ├── layout.tsx               # Admin shell layout
│   │   ├── login/page.tsx           # Login page
│   │   └── pipeline/               # Pipeline monitoring dashboard
│   │       ├── page.tsx
│   │       └── pipeline-charts.tsx
│   ├── layout.tsx                   # Root layout (RootProvider, fonts, metadata)
│   ├── global.css                   # Shared docs design system and token layers
│   ├── docs/[[...slug]]/
│   │   ├── page.tsx                 # Docs page renderer (hero + MDX body + context rail)
│   │   └── layout.tsx               # Docs sidebar layout (DocsLayout, nav links)
│   └── docs-og/[[...slug]]/route.tsx # Dynamic OG image generation
├── components/
│   ├── mdx.tsx                      # MDX component registry (getMDXComponents)
│   ├── mdx/
│   │   ├── mermaid.tsx              # Zoomable Mermaid renderer (zoom/pan, theme-aware)
│   │   ├── sql-playground.tsx       # DuckDB-WASM SQL sandbox
│   │   └── observable-plot.tsx      # ObservablePlot, ShotChart, GameFlow, PlayerCompare,
│   │                                #   SeasonTrend, DistributionPlot, HeatmapGrid
│   ├── site/
│   │   ├── brand-mark.tsx           # Canonical docs logo mark
│   │   ├── counter.tsx              # Animated count-up (IntersectionObserver)
│   │   ├── docs-shell.tsx           # Barrel export for docs chrome
│   │   ├── docs-generated-*.tsx     # Generated-page entry, coverage, scan, and modules
│   │   └── docs-*.tsx               # Docs chrome pieces (nav, hero, context rail)
│   ├── ui/
│   │   ├── badge.tsx                # CVA badge (6 variants)
│   │   ├── button.tsx               # CVA button (4 variants, 4 sizes, asChild)
│   │   ├── card.tsx
│   │   ├── tabs.tsx
│   │   └── skeleton.tsx
│   └── admin/                       # 13 admin dashboard components
│       ├── admin-shell.tsx, admin-nav.tsx
│       ├── kpi-card.tsx, sparkline-card.tsx, tracker-bar.tsx
│       ├── chart-area.tsx, chart-bar.tsx, chart-donut.tsx
│       ├── bar-list.tsx, content-freshness.tsx, freshness-heatmap.tsx
│       ├── status-dot.tsx, data-table.tsx
├── content/docs/                    # MDX content (7 sections, 51 pages)
│   ├── meta.json                    # Root nav ordering (Getting Started / Reference / Guides)
│   ├── index.mdx                    # Docs landing
│   ├── installation.mdx, architecture.mdx, cli-reference.mdx
│   ├── playground.mdx               # DuckDB-WASM SQL sandbox page
│   ├── schema/                      # Star schema reference (9 pages)
│   ├── data-dictionary/             # Field-level documentation (6 pages including glossary)
│   ├── diagrams/                    # ER, pipeline, endpoint diagrams (5 pages)
│   ├── endpoints/                   # API endpoint documentation (8 pages)
│   ├── lineage/                     # Data lineage traces (4 pages; machine JSON lives in lib/generated/)
│   └── guides/                      # User and operator guides (14 pages)
├── lib/
│   ├── site-config.ts               # Barrel export for split site-config modules
│   ├── site-metrics.generated.ts    # Auto-generated homepage scoreboard metrics
│   ├── source.ts                    # Content loader (fumadocs-core/source)
│   ├── duckdb.ts                    # DuckDB-WASM singleton, query runner, Parquet loader
│   ├── use-zoom-pan.ts              # Zoom/pan hook for Mermaid diagrams
│   ├── utils.ts                     # cn() className helper, breadcrumb utils
│   └── admin/                       # Admin data fetchers and types
│       ├── pipeline.ts
│       └── types.ts
├── proxy.ts                         # Admin auth guard / request interception (HMAC session cookie, 24h TTL)
├── source.config.ts                 # fumadocs-mdx config (remarkMdxMermaid plugin)
├── next.config.mjs                  # Next.js config (createMDX wrapper)
└── package.json
```

## 3. Navigation

- Sidebar ordering is controlled by `meta.json` files in each content directory
- Root `meta.json` defines three section groups with `---Separator---` syntax:
  - **Getting Started** — index, installation, architecture, cli-reference
  - **Reference** — schema, data-dictionary, diagrams, endpoints, lineage
  - **Guides** — playground, guides
- Subsection `meta.json` uses `pages` arrays with `---` separators for ordering
- `guides/meta.json` groups content into **Onboarding**, **Tutorials**, **Operations**, **Troubleshooting**, and **Maintainers**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wyattowalsh/nbadb](https://github.com/wyattowalsh/nbadb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
