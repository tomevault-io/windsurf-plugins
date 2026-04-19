---
trigger: always_on
description: Component organization patterns - when to break out into folders, where to put constants and types
---


# Frontend Component Organization Rules

## When to Break Out into a Folder

Break a component into its own folder when **any** of these conditions are met:

1. **Multiple related components** - Component has 3+ child components or sub-components
2. **Shared constants/types** - Component has constants or types that are used by multiple child components
3. **Complex component** - Component file exceeds 300-400 lines and has clear logical sections
4. **Reusable sub-components** - Component has sub-components that might be reused elsewhere
5. **Multiple related files** - Component needs utilities, hooks, or helpers specific to it

### Examples

✅ **BREAK OUT** - DashboardLayout has multiple components (NavItem, PageHeader, Sidebar, etc.)
```
layouts/
  DashboardLayout/
    DashboardLayout.tsx    # Main component
    NavItem.tsx           # Sub-component
    PageHeader.tsx         # Sub-component
    Sidebar.tsx            # Sub-component
    PageContent.tsx        # Sub-component
    HeaderWithOrderDate.tsx # Sub-component
    constants.ts           # Shared constants
    types.ts              # Shared types
    index.ts              # Public exports
```

✅ **BREAK OUT** - BuyerOps container has multiple tables, nested sub-folders, and extracted hooks
```
containers/
  BuyerOps/
    index.tsx                   # Main container entry point
    BuyerOpsContent.tsx         # Orchestration component
    BuyerOpsDataLoader.tsx      # Data loading wrapper
    constants.ts                # Shared constants (transform IDs, filter text)
    types.ts                    # Shared types (BuyerOpsParams, etc.)
    utils.ts                    # Shared utility functions
    hooks/                      # ← Hooks subdirectory for BuyerOps-level hooks
      index.ts                  # Re-exports all hooks
      useBuyerOpsSave.ts        # Save/cancel/edit-state management
    FilterBar/                  # ← Nested component folder (filter controls)
      index.ts                  # Barrel — public API (exports FilterBar only)
      FilterBar.tsx             # Main filter bar component
      FilterSelect.tsx          # FilterSelect + StatusFilterSelect sub-components
      utils.ts                  # Filter handler helpers (getFilterHandler, etc.)
    ProductTable/               # ← Nested component folder
      index.ts                  # Barrel — public API for external consumers
      ProductTable.tsx          # Main table component (composes sub-components)
      ProductGroupRow.tsx       # Row renderer for group rows
      ProductDetailRow.tsx      # Row renderer for detail/expansion rows
      SummaryTable.tsx          # PO-level summary table
      EditAllDialog.tsx         # Bulk-edit dialog
      BatchesEditableCell.tsx   # Editable cell for batches
      EditableCell.tsx          # Generic editable cell
      constants.ts              # Column definitions, numeric columns
      types.ts                  # ItemData, FlattenedRow, SummaryData
      utils.ts                  # Key-generation helpers
      hooks/                    # ← Hooks grouped in subdirectory
        index.ts                # Re-exports all hooks
        useBuyerOpsEdits.ts     # Edit state machine
        useEditTracking.ts      # Per-row edit counters + global count
        useScrollPreservation.ts # Virtuoso scroll + expand/collapse restoration
        useSummaryData.ts       # Summary computation
      SupplyWalkTable/          # ← Nested sub-component folder
        index.ts                # Public exports (component + cache + types)
        SupplyWalkTable.tsx
        SupplyWalkTableHeader.tsx
        SupplyWalkTableRow.tsx
        ExpirationWarningBanner.tsx
        supplyWalkCache.ts      # Cache singleton lives with its feature
        constants.ts            # supplyWalkTheme
        types.ts                # SupplyWalkCacheEntry, SupplyWalkCacheRef, SupplyWalkTableProps
        utils.ts                # formatValue, getCellStyle, buildRecalculationInfo, sortRowsByMeasure
        hooks/                  # ← Hooks grouped in subdirectory
          index.ts              # Re-exports all hooks
          useSupplyWalkCache.ts       # Cache management for Virtuoso remounts
          useSupplyWalkRecalculation.ts # Recalculation when CORRECTED_BATCHES changes
    SupplierTable/              # ← Nested component folder
      index.ts                  # Barrel — public API
      SuppliersTable.tsx        # Main table component
      SupplierTableRow.tsx      # Row renderer with diff formatting
      constants.ts              # COLUMN_ORDER, NUMERIC_COLUMNS
      types.ts                  # SuppliersTableProps, SupplierRow
      hooks/                    # ← Hooks always in subdirectory
        index.ts                # Re-exports all hooks
        useScrollPreservation.ts # Virtuoso scroll position preservation
    POTable/                    # ← Nested component folder
      index.ts                  # Barrel — public API
      POTable.tsx               # Main table component
      TableRow.tsx              # Row renderer
      constants.ts              # COLUMN_ORDER, STATUS_CONFIG
      types.ts                  # POTableProps, SupplierLocationRow, WarningFlags

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atomic-supply) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
