---
trigger: always_on
description: You will help the user build out this React-based web app that visualizes data from Power BI semantic models. The app fetches live data via DAX queries, renders charts and grids using Vega-Lite and a built-in DataGrid component, and supports light/dark theming. Your job is to discover the user's data, write correct DAX queries, build React components that fetch and display that data, and validate the result in the browser.
---

# Agent Instructions

## Purpose

You will help the user build out this React-based web app that visualizes data from Power BI semantic models. The app fetches live data via DAX queries, renders charts and grids using Vega-Lite and a built-in DataGrid component, and supports light/dark theming. Your job is to discover the user's data, write correct DAX queries, build React components that fetch and display that data, and validate the result in the browser.

## Sub-Agent Delegation

Break tasks into independent pieces and delegate them to sub-agents running in parallel. Don't do work sequentially when it can be done concurrently.

For example, when building a new dashboard page: a sub-agent finds the semantic model and discovers its schema. Separate sub-agents write the DAX query files, then have separate sub-agents build each component in parallel once the queries are ready.

## Project Structure

```
fabric.yaml                # Fabric connection config (managed by `npx fabric-app-data`)
index.html                 # Vite entry HTML
vite.config.ts             # Vite + Tailwind build config
tsconfig.json              # TypeScript configuration
src/
├── fabric.generated.ts    # Auto-generated from fabric.yaml — connection aliases → workspace/item IDs
├── main.tsx               # App entry point
├── App.tsx                # Main dashboard layout
├── ErrorFallback.tsx      # Error boundary fallback UI
├── global.css             # Tailwind v4 @theme design tokens
├── components/            # Dashboard UI components (cards, charts, banners)
├── hooks/                 # React hooks (data fetching, theming)
├── lib/                   # Utilities, Fabric client
├── queries/               # DAX queries (.dax) + Vega-Lite specs (.json) + factory functions (.ts), grouped by page/domain
└── vite-env.d.ts          # Vite type declarations
```

### Query & Spec Organization

DAX queries and Vega-Lite specs live in `src/queries/`, grouped by dashboard page or domain. Each visualization gets files sharing the same kebab-case base name: one or more `.dax` files for queries, a `.json` file for the Vega-Lite spec, and a `.ts` factory file that imports them and exports `{ connection, query, columnMetadata, vegaLiteSpec }`. The factory function accepts optional parameters to select between query variants or modify the spec:

```
src/queries/
├── index.ts                            # Re-exports all query modules
├── {page-or-domain}/                   # Group by dashboard page or domain
│   ├── {visualization-name}.dax        # DAX query (plain text)
│   ├── {visualization-name}-{variant}.dax  # Additional query variants (optional)
│   ├── {visualization-name}.json       # Vega-Lite spec (JSON)
│   ├── {visualization-name}.ts         # Factory function: imports .dax + .json, exports { connection, query, vegaLiteSpec, columnMetadata }
│   └── index.ts                        # Re-exports all visualizations in this group
```

#### Example TS File

**`revenue-by-region.ts`** — the factory function accepts use-case-specific parameters and uses them to modify the DAX query and/or Vega-Lite spec as appropriate:
```ts
import type { VisualizationSpec } from "@microsoft/fabric-visuals";
import type { ColumnMetadataMap } from "@/lib/to-data-table";
import baseQuery from "./revenue-by-region.dax?raw";
import spec from "./revenue-by-region.json";

const connection = "{connection-alias}";  // from fabric.yaml

/** Column metadata keyed by original DAX column name. */
const columnMetadata: ColumnMetadataMap = {
  "Products[Region]": { name: "ProductsRegion", displayName: "Region" },
  "[Total Revenue]": { name: "Total Revenue", displayName: "Total Revenue", format: "$#,0.00" },
};

interface RevenueByRegionParams {
  /** Filter to specific product categories (modifies the DAX query). */
  categories?: string[];
  /** Only show regions with revenue above this threshold (modifies the Vega-Lite spec). */
  minRevenue?: number;
}

export function revenueByRegion(params?: RevenueByRegionParams) {
  let query = baseQuery;
  let vegaLiteSpec = spec; // Should clone if modifying

  if (params?.categories?.length) {
    // make changes to the DAX query to filter by the specified categories and update the query variable
  }

  if (params?.minRevenue != null) {
    // Clone spec and append a client-side filter transform to the Vega-Lite spec
  }

  return { connection, query, columnMetadata, vegaLiteSpec };
}
```

The parameters, their types, and how they modify the query or spec are **entirely use-case-specific**. Some visualizations may need no parameters at all; others may accept date ranges, top-N limits, grouping dimensions, or string search terms. The factory function is the single place that translates caller intent into DAX and/or Vega-Lite modifications.

#### Query variants with multiple `.dax` files


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fredgis/FabricAtlas](https://github.com/fredgis/FabricAtlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
