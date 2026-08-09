---
trigger: always_on
description: enableSorting: true,
---

# Migrating Tables from mantine-react-table to mantine-datatable

This document captures every pattern, fix, and decision from the downloads widget rework (PR #6430). Use it as a playbook when porting other tables.

## Migration Candidates

Current `mantine-react-table` consumers that could be migrated:

**Widgets (in `packages/widgets/`):**
- `docker/component.tsx` — Docker container table
- `media-server/component.tsx` — Media server sessions

**Admin pages (in `apps/nextjs/src/app/[locale]/manage/`):**
- `tools/docker/docker-table.tsx`
- `tools/tasks/_components/tasks-table.tsx`
- `tools/api/components/api-keys.tsx`
- `users/_components/user-list.tsx`
- `users/invites/_components/invite-list.tsx`
- `tools/kubernetes/` — 8 Kubernetes resource tables

**Modals:**
- `packages/modals-collection/src/search-engines/request-media-modal.tsx`

**Shared hook to remove once all migrated:**
- `packages/ui/src/hooks/use-translated-mantine-react-table.ts`

---

## 1. Core Migration: mantine-react-table → mantine-datatable

### Import changes

```typescript
// BEFORE
import { MantineReactTable, useMantineReactTable } from "mantine-react-table";
import type { MRT_ColumnDef } from "mantine-react-table";

// AFTER
import { DataTable, useDataTableColumns } from "mantine-datatable";
import type { DataTableColumn, DataTableSortStatus } from "mantine-datatable";
```

### Column definition shape

```typescript
// BEFORE (MRT)
const columns: MRT_ColumnDef<MyRow>[] = [
  {
    accessorKey: "name",
    header: "Name",
    size: 200,
    enableSorting: true,
    Cell: ({ row }) => <Text>{row.original.name}</Text>,
  },
];

// AFTER (mantine-datatable)
const columns: DataTableColumn<MyRow>[] = [
  {
    accessor: "name",           // not accessorKey
    title: "Name",              // not header
    width: 200,                 // not size
    sortable: true,             // not enableSorting
    render: (record) => <Text>{record.name}</Text>,  // not Cell
  },
];
```

Key differences:
- `accessorKey` → `accessor`
- `header` → `title`
- `size` → `width`
- `Cell: ({ row }) =>` → `render: (record) =>`
- `enableSorting` → `sortable`
- `enableColumnActions` → does not exist (no column action menus)
- `mantineTableBodyCellProps` → `cellsStyle` in `defaultColumnProps`

### Table component

```typescript
// BEFORE
const table = useTranslatedMantineReactTable({ columns, data, ... });
return <MantineReactTable table={table} />;

// AFTER
return (
  <DataTable
    records={data}
    columns={columns}
    sortStatus={sortStatus}
    onSortStatusChange={setSortStatus}
    ...
  />
);
```

### Sorting is manual

mantine-datatable does NOT sort for you. You must sort the data yourself:

```typescript
const [sortStatus, setSortStatus] = useState<DataTableSortStatus<MyRow>>({
  columnAccessor: "name",
  direction: "asc",
});

const sortedData = useMemo(() => {
  const mult = sortStatus.direction === "desc" ? -1 : 1;
  return [...data].toSorted((a, b) => {
    const aVal = a[sortStatus.columnAccessor];
    const bVal = b[sortStatus.columnAccessor];
    if (typeof aVal === "string") return aVal.localeCompare(bVal as string) * mult;
    return ((aVal as number) - (bVal as number)) * mult;
  });
}, [data, sortStatus]);

// Pass sortedData to records, not data
<DataTable records={sortedData} sortStatus={sortStatus} onSortStatusChange={setSortStatus} />
```

---

## 2. Making Tables Transparent (Widget Context)

Widgets sit inside board cards that have user-controlled opacity. The table must be fully transparent so the card background shows through. mantine-datatable defaults to `var(--mantine-color-body)` which is opaque.

Create a CSS file alongside the component:

```css
/* styles.css */
.my-table,
.my-table .mantine-datatable-table,
.my-table .mantine-datatable-table thead,
.my-table .mantine-datatable-table tbody,
.my-table .mantine-datatable-table tfoot,
.my-table th,
.my-table td {
  background-color: transparent !important;
}

.my-table .mantine-datatable-table tr {
  background-color: transparent;
}

/* Hover: translucent instead of opaque */
.my-table .mantine-datatable-table tbody tr:hover {
  background-color: color-mix(in srgb, var(--mantine-color-default-hover) 40%, transparent) !important;
}

/* Headers: frosted glass effect */
.my-table th {
  white-space: nowrap;
  background-color: color-mix(in srgb, var(--mantine-color-body) 60%, transparent) !important;
  backdrop-filter: blur(8px);
}

/* Cell overflow */
.my-table td {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Hide sort icon until hover */
.my-table .mantine-datatable-header-cell-sortable-icon {
  opacity: 0;
  transition: opacity 150ms;
}
.my-table th:hover .mantine-datatable-header-cell-sortable-icon {
  opacity: 1;
}

/* Row expansion area */
.my-table .mantine-datatable-row-expansion-cell {
  background-color: color-mix(in srgb, var(--mantine-color-body) 40%, transparent) !important;
}
```

Import it: `import "./styles.css";` and apply via `className="my-table"`.

---

## 3. Responsive Column Visibility

Use a lookup table of width breakpoints instead of media queries. The widget receives `width` as a prop.

```typescript
interface SizeConfig {
  fontSize: "xs" | "sm";
  iconSize: number;
  cellPadding: number;
  showSpeedColumns: boolean;
  showTimeColumn: boolean;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homarr-labs/homarr](https://github.com/homarr-labs/homarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
