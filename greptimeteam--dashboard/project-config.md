---
trigger: always_on
description: Greptime dashboard UI tokens, layout regions, tables, and Arco overrides — use when editing Vue/LESS styles
---


# Dashboard UI style rules

## Core principles

1. Use **design tokens only**: `--gpt-*`, `--font-family-base`, `--font-mono`. Do not add `#hex`, raw `12px`, or legacy `--color-text-*` / `--main-font-color` in new code.
2. Prefer **global CSS classes** in `ui-regions.less`, `typography.less`, `button.less`, `dataView.less`. Vue scoped styles are for **page semantics only** (column emphasis, chart, pipeline colors).
3. **No layout Vue wrappers** (`PageHeader.vue`, etc.). Compose modifier classes on existing DOM.
4. **CSS selector = template class**: if scoped/CSS uses `.foo.gpt-bar`, the template must have `class="foo gpt-bar"`.
5. Query results use **`QueryResultTable` → `PaginatedDataTable` → `DataTable`**. Do not reintroduce `data-grid.vue` or standalone `a-table` for SQL results.

## Layout modifiers (pick one on page root)

| Modifier | Pages | Table mode |
|----------|-------|------------|
| `new-layout new-layout--workspace` | Table Query, Metrics, Perses | Panel: scroll inside table, pagination fixed at bottom |
| `query-layout query-layout--stack` | Logs Query, Traces | Last card `flex:1`; toolbar via `.toolbar` in `ui-regions.less` |
| `query-layout query-layout--surface` | Flow | Page scroll; `gpt-table-page` + `scroll.y: undefined` |

Files: `src/assets/style/ui-regions.less`, `src/assets/style/query-layout.less`.

## Region classes (put on template)

- **Header**: `gpt-page-header` on `.page-header` (do not duplicate page-header block in `query-layout.less`).
- **Toolbars**: `gpt-toolbar` / `gpt-content-toolbar` / `gpt-results-toolbar` (results = gray table toolbar bg).
- **Query strip** (workspace): `gpt-query-strip`.
- **Results card** (workspace): `gpt-results-pane` on result `a-card`.
- **Panel table** (internal scroll): `gpt-table-panel` on flex column container.
- **Page table** (document scroll): `gpt-table-page` on `DataTable`.
- **Surface card**: `gpt-surface-card` + `gpt-surface-card__title` / `__badge` in `#title`.
- **Editors**: `gpt-light-editor`, `gpt-light-editor-card`, `gpt-code-editor-frame` (+ `--h200/300/500`), `gpt-square-editor` when needed.
- **Misc**: `gpt-divider-band`, `gpt-popover-panel`, `gpt-card-flush-body`, `gpt-status-pill` (+ `is-running` / `is-error` / `is-default`), `gpt-status-pill__dot`.

Typography utilities: `typography.less` — e.g. `gpt-text-page-subtitle`, `gpt-text-secondary`, `gpt-text-muted`, `gpt-text-toolbar`, `gpt-text-toolbar-meta`, `gpt-icon-muted`.

## Toolbar typography

Base font in toolbars is **`--gpt-font-base` (12px)**. Weight follows **role**, not location:

| Role | Weight | Color (idle) | Examples |
|------|--------|----------------|----------|
| **Control** (peer actions) | **`--gpt-font-weight-control` (600)** | `--gpt-text-primary`, or **secondary** for button-radio idle only | `a-button`, `TimeRangeSelect`, **Live** (`gpt-toolbar-checkbox`), Builder/Code radio |
| **Plain label** | `--gpt-font-weight-regular` (400) | `--gpt-text-primary` | `gpt-text-toolbar` on non-interactive copy |
| **Meta / hint** | 600 | `--gpt-text-secondary` or `--gpt-text-muted` | Row counts, `gpt-text-toolbar-meta`, **Columns** text-btn label |

**600 on top toolbar is correct** — all Arco buttons use `@btn-font-weight: 600` (`arco-theme.less`). Inline toolbar checkboxes must match via `gpt-toolbar-checkbox`, not default `.arco-checkbox-label` (500).

Do **not** put `gpt-text-secondary` on peer toolbar toggles (Live, Run neighbors). Secondary is for **unselected** button-radio segments and **meta** text only.

Regions: `.gpt-toolbar`, `.gpt-content-toolbar`, `.query-layout .toolbar` (`ui-regions.less`).

## Panel table height chain (Table Query)

```
.result-container.gpt-table-panel
  .result-toolbar.gpt-results-toolbar
  .result-content (flex:1; min-height:0; overflow:hidden)
    .table-panel → QueryResultTable → .paginated-data-table
      .data-table-container (flex:1; scroll inside)
      .grid-pagination (flex-shrink:0; fixed at bottom)
```

Tab pane: `overflow: hidden` on content item (avoid outer scroll stealing from table). `data-view/index.vue` is the reference.

## Tables & font size

- **`DataTable` default `size`: `medium`** (14px body via Arco). Do not default to `small` for main result tables.
- `dataView.less` forces **12px** on `.arco-table-size-small` / `mini` cells — use only when density is intentional (compact mode).
- Logs: `medium` normal, `mini` when compact checkbox on.
- Flow list: `gpt-table-page`, no virtual fill height.

## Color system (`global.less` + `arco-theme.less`)

**Source of truth**: define new colors only in `:root` (`global.less`) or Arco compile tokens (`arco-theme.less`). Component/page CSS references tokens — no new `#hex` / `rgb()` in Vue/LESS except chart canvas / vp-doc (see exceptions).

### Text hierarchy (four tiers — not all main-dark)

| Tier | Token | Hex | Use |
|------|--------|-----|-----|
| Primary | `--gpt-text-primary` | `#473460` | Page titles, input/select **values**, table body, outline/text button idle, standard checkbox label |
| Secondary | `--gpt-text-secondary` | `#8b7ba8` | Toolbar meta, button-radio **idle**, table header, **Columns** / pagination idle |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GreptimeTeam/dashboard](https://github.com/GreptimeTeam/dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
