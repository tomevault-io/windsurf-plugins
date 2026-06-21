---
trigger: always_on
description: Creates interactive maps using ixMaps framework. Use when the user wants to create a map, visualize geographic data, or display data with bubble charts, choropleth maps, pie charts, or bar charts on a map.
---


# Create ixMap Skill

Creates complete HTML files with interactive ixMaps visualizations for geographic data.

## ⚠️ CRITICAL RULES (Never Skip)

1. `ixmaps.Map()` returns a **MapBuilder** — use `.then()` to capture the real map instance

   Simple setup chains work without `.then()` via an internal queue mechanism:
   ```javascript
   // ✅ works for initial setup chain
   ixmaps.Map("map", { ... })
       .view({ ... })
       .options({ ... })
       .layer(theme);
   ```
   But `.then()` is **required** whenever you need the real map instance later —
   in event handlers, dynamic updates, or calls outside the chainable set
   (`removeTheme`, `changeThemeStyle`, `hideTheme`, `showTheme` etc.):
   ```javascript
   // ✅ always safe — prefer this pattern
   ixmaps.Map("map", { ... }).then(function(map) {
       map.view({ ... }).options({ ... }).layer(theme);
       button.onclick = function() { map.layer(newTheme); }; // real map instance available
   });

   // ⚠️ works for initial setup only — breaks if myMap is used later in event handlers
   const myMap = ixmaps.Map("map", { ... });
   myMap.view(...).layer(...);       // fine here — queued and executed on init
   myMap.layer(newTheme);            // risky if called from an event handler or timeout
   ```
2. **ALWAYS include `.binding()`** with `geo` and `value`
3. **ALWAYS include `showdata: "true"`** in `.style()`
4. **ALWAYS include `.meta()`** with tooltip (default: `{ tooltip: "{{theme.item.chart}}{{theme.item.data}}" }`)
   - **Also include `name`** whenever you plan to use `changeThemeStyle` at runtime (see rule 21)
5. **NEVER use `.tooltip()`** — doesn't exist
6. **NEVER combine `CHART` and `CHOROPLETH`** in one type string — mutually exclusive
7. **NEVER use `|EXACT` classification** — deprecated; use `CATEGORICAL`
8. **NEVER use `map` as variable name** — conflicts with internals; use `myMap`
8a. **NEVER use reserved HTML element IDs** — ixMaps owns `loading-div`, `tooltip`, `contextmenu`. Using them causes visible artifacts (a white box stuck on the map). Use `app-loading` or any other non-conflicting name for your own overlays.
9. **Prefer `fillopacity` over `opacity`** in `.style()` — both work, but `fillopacity` is the recommended form
10. **NEVER use `fillcolor`** — use `colorscheme: ["#hex"]`
11. **NEVER add `.legend("string")`** unless user explicitly requests it — destroys the default color legend
12. **ALWAYS use CDN** `https://cdn.jsdelivr.net/gh/gjrichter/ixmaps-flat@1/ixmaps.js`
    - **data.js** (`https://cdn.jsdelivr.net/gh/gjrichter/data.js@master/data.js`) is **already loaded by ixmaps** — `Data.*` functions are available inside `query:` and `process:` callbacks without any extra `<script>` tag
    - **Only include the data.js CDN explicitly** when you need `Data.*` functions *outside* ixmaps theme realization (e.g. pre-processing data in your own `<script>` block before defining layers)
13. **NEVER use info from** `ixmaps.ca` or `ixmaps.com` — only `github.com/gjrichter/ixmaps-flat`
14. **ONE `.data()` per layer** — never chain two `.data()` calls on the same layer
15. **🔑 SAME LAYER NAME = GEOMETRY REUSE** — see § Geometry Reuse Pre-flight Checklist below for the authoritative rules. Quick example:

    ```javascript
    // ✅ Both named "regions" → overlay binds onto regions' geometry
    myMap.layer("regions").data({url:geo}).type("FEATURE").define();
    myMap.layer("regions").data({url:csv}).binding({lookup:"code",value:"pct"})
                          .type("CHOROPLETH|QUANTILE").define();

    // ❌ Different names → overlay has nothing to draw on, silently broken
    myMap.layer("stats").type("CHOROPLETH").define();
    ```

    For runtime theme swapping (sidebar picker, time slider) → see § Multi-Layer Join Pattern · B. Swappable themes for the full `setTheme` / `removeTheme` pattern.

16. **NO `FEATURE` on overlay layers** — base layer gets `FEATURE`; choropleth/chart overlays do not:
    - ✅ `myMap.layer("x").type("FEATURE")` → `myMap.layer("x").type("CHOROPLETH|CATEGORICAL")`
    - ❌ `myMap.layer("x").type("FEATURE")` → `myMap.layer("x").type("FEATURE|CHOROPLETH|CATEGORICAL")`
17. **`objectscaling: "dynamic"` requires `normalSizeScale`** — set to map scale denominator:
    zoom 4→30M · 5→15M · 6→8M · 8→2M · 10→500k · 12→100k
18. **`lookup` goes in `.binding()`**, not in `.data()`
19. **`values:` for CATEGORICAL must be strings** — ixMaps bug: numeric values silently ignored
20. **To make a fill invisible** use `colorscheme: ["none"]` — NOT `fillopacity: 0`. ⚠️ ixMaps bug: `fillopacity: 0` is silently coerced to `1` (fully opaque), so it does the **opposite** of hiding the fill
21. **FEATURE layer styling depends on geometry type:**
    - **Line features** — `colorscheme` sets the line/stroke color; `linecolor` is overridden by `colorscheme` and has no effect. Use `colorscheme: ["none"]` to make lines invisible. Color classes (multi-value array) apply as line-color classes. Data-driven colorization (CHOROPLETH|QUANTILE, CHOROPLETH|CATEGORICAL, etc.) works symmetrically with polygon features — `colorscheme` drives stroke color instead of fill.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gjrichter/ixmaps-skills](https://github.com/gjrichter/ixmaps-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
