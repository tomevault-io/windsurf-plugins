---
trigger: always_on
description: Project: the-datagrid
---

Project: the-datagrid

Mission
the-datagrid is a React DataGrid library that provides an Inovua-like developer experience (API shape + type naming) while keeping the implementation lightweight and maintainable. The library prioritizes stability of its public API and type vocabulary over adding new knobs and options.

Non-negotiable public contract (React component props)
ReactDataGrid MUST support exactly these props as the public instantiation surface:

* theme
* idProperty
* columns
* dataSource
* columnOrder
* enableColumnFilterContextMenu
* enableColumnAutosize
* skipHeaderOnAutoSize
* enableFiltering
* defaultFilterValue
* filteredRowsCount
* onColumnOrderChange
* virtualized
* columnUserSelect
* i18n
* showColumnMenuTool

Rules:

1. Do not introduce new public props without an explicit decision. If functionality cannot fit into the fixed prop surface, it must be implemented internally, through column-level configuration (TypeColumn fields), or deferred.
2. Do not rely on consumers passing additional props “for styling” or “for layout”. Styling must be handled internally via Tailwind/shadcn conventions (see below).
3. Maintain backward compatibility for the semantics of these props once released.

Canonical exported types (Inovua-aligned vocabulary)
the-datagrid exposes a naming and conceptual model aligned with Inovua, even if the implementation is simplified:

* IColumn, TypeColumn, TypeColumns
* TypeDataGridProps
* TypeDataSource
* TypeFilterValue, TypeSingleFilterValue
* TypeSortInfo, TypeSingleSortInfo, SortDirection
* TypeFilterTypes, TypeFilterType, TypeFilterOperator
* TypeI18n

The goal is migration ergonomics: a developer familiar with Inovua should understand the types immediately, and app-level code should need minimal rewrites.

Behavior contracts

1. Columns

* Every column must have a stable identifier via id or name.
* columnOrder is a list of column IDs (derived from id/name) and defines the rendered order.
* Column visibility is controlled by column.visible (if supported) but must not require new public props.
* Column rendering must support header/renderHeader/render and alignments (headerAlign/textAlign).
* Column sizing must support width/defaultWidth/minWidth/maxWidth and work with enableColumnAutosize.

2. DataSource (local + remote)
   TypeDataSource must support:

* Local array data sources: any[]
* Remote sources:

  * Promise<any[]>
  * Promise<{ data: any[]; count: number }>
  * (args) => any[] | Promise<any[]> | Promise<{ data: any[]; count: number }>

Remote args must include, at minimum:

* sortInfo
* filterValue
* columnOrder
* columns
* idProperty
* theme
  This args object is part of the API contract and must remain stable.

3. Filtering

* enableFiltering toggles the filter row behavior.
* defaultFilterValue initializes uncontrolled filter state.
* Filter state model uses TypeFilterValue = TypeSingleFilterValue[] | null.
* Each filter entry includes name (column key), operator, type, value, and optional active.
* enableColumnFilterContextMenu enables operator changes via a context menu on the filter cell.
* For local array sources: filtering must be applied client-side.
* For remote sources: filterValue must be passed to dataSource(args).
* filteredRowsCount must report the number of rows after filtering (and before any local pagination slicing if pagination exists internally).

4. Sorting

* Sorting state uses TypeSortInfo = single | array | null.
* Toggle behavior must be deterministic and respect allowUnsort semantics internally (even if allowUnsort is not publicly configurable).
* For local array sources: sorting must be applied client-side.
* For remote sources: sortInfo must be passed to dataSource(args).

5. Column order + reordering

* columnOrder + onColumnOrderChange define the ordering contract.
* Column reordering must only commit changes through onColumnOrderChange (no hidden mutation of consumer state).
* If onColumnOrderChange is not provided, reordering should be disabled (the grid can render columnOrder but must not pretend it can persist changes).

6. Virtualization

* virtualized controls row virtualization (TanStack Virtual or equivalent).
* Virtualization must not break:

  * header rendering
  * filter row rendering
  * column widths/autosizing behavior
  * context menus positioning

7. Autosizing

* enableColumnAutosize enables a deterministic width heuristic.
* skipHeaderOnAutoSize controls whether header text is included in measurement.
* Autosize should use a bounded sampling strategy (e.g., first N rows) and clamp widths to sane limits.
* Avoid relying on DOM measurement where possible; prefer deterministic estimation (fast, SSR-friendly).

8. i18n

* i18n is an object map keyed by known UI keys, with fallbacks.
* At minimum, cover: noRecords, clear, clearAll, contains/startsWith/endsWith/eq/neq/empty/notEmpty, columns, sortAsc/sortDesc/unsort.

Tailwind + shadcn: mandatory design system alignment (why it matters and how to implement it)

This is not optional. the-datagrid must “feel native” inside a shadcn + Tailwind application, because that is the primary integration environment. If the grid ships with bespoke styling or mismatched UI primitives, it becomes visually inconsistent, harder to theme, and expensive to maintain across products.

Design requirements:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geo-vi/the-datagrid](https://github.com/geo-vi/the-datagrid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
