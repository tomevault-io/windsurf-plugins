---
trigger: always_on
description: import { NeoDroppableZone, NeoSortableItem, NeoSortableProvider, NoDraggable } from '@dvcol/neo-svelte/sortable';
---

# `@dvcol/neo-svelte/sortable`

```ts
import { NeoDroppableZone, NeoSortableItem, NeoSortableProvider, NoDraggable } from '@dvcol/neo-svelte/sortable';
```

## Components

- `NeoSortableProvider` — context root. Creates a `NeoSortableContext`, wires up `@dnd-kit`'s `DragDropProvider`, and passes the context to a `children` snippet. Accepts both a **single-list** (`NeoSortableItem<Data>[]`) and a **multi-list** (`Record<UniqueIdentifier, NeoSortableItem<Data>[]>`) `items` shape. Must wrap all sortable items and droppable zones.
- `NeoSortableItem` — a single sortable entry. Must be a descendant of `NeoSortableProvider`. Registers itself in the context on mount and deregisters on destroy. Passes an `instance` (the `createSortable` return value) to the `children` snippet; apply `{@attach instance.attach}` to the host DOM element.
- `NeoDroppableZone` — a standalone droppable area with no sort logic. Use as a fallback drop target for empty containers in a multi-list layout. Passes the droppable `instance` to `children`.
- `NoDraggable` — a standalone, **non-sortable** draggable source (for dragging an item into a `NeoDroppableZone`, not for reordering within a list). Passes the draggable `instance` to `children`.

## Concepts

### `items` shape

`NeoSortableProvider` accepts two shapes for `items`:

- **Array** (`NeoSortableItem<Data>[]`) — single-list drag-to-reorder.
- **Record** (`Record<UniqueIdentifier, NeoSortableItem<Data>[]>`) — multi-list drag between containers. Each key is a container id; `@dnd-kit/helpers`'s `move` handles cross-list item transfer automatically.

`items` is `$bindable`; always write `bind:items` so the parent array/record stays in sync after every drag.

### Attach directive

After `createSortable` / `createDraggable` / `createDroppable` returns an instance, wire it to the DOM element via `{@attach instance.attach}`. Without the attach the element has no registered position with `@dnd-kit` and dragging silently fails.

For drag-handle sub-elements (restrict drag activation to a child handle), use `{@attach instance.attachHandle}` on that child and `{@attach instance.attach}` on the outer item element. `NeoHandle` (from `@dvcol/neo-svelte/floating`) exposes both via `{@attach instance.attachHandle}`.

### `axis` restriction

`axis="x"` / `axis="y"` applies the corresponding `@dnd-kit` modifier. Defaults to unrestricted (any direction). Useful for purely horizontal or purely vertical lists.

### `container` prop

The optional `container` prop (`HTMLElement`) restricts drag movement to that element's bounds via `RestrictToElement`. Defaults to `document.body`. Typical pattern: declare a template variable, bind it with `bind:this`, and pass it as the container:

```svelte
<script lang="ts">
  let listEl = $state<HTMLElement>();
</script>

<NeoSortableProvider {container} bind:items>
  {#snippet children(ctx)}
    <ol bind:this={listEl} data-size={ctx.items.length} ...>...</ol>
  {/snippet}
</NeoSortableProvider>
```

### Drag overlay

Pass an `overlay` snippet to `NeoSortableProvider` to activate a `DragOverlay` — a portal-rendered clone that follows the cursor during drag. Without it the item itself translates. When the overlay is active, hide the in-place item while it is being dragged:

```svelte
<li data-grabbed={instance.isDragging} style:opacity={instance.isDragging ? 0 : 1}>
  ...
</li>
```

### Cancel / snapshot

`NeoSortableContext` snapshots `items` on `dragStart`. If the drag is cancelled (Escape key or externally via `event.canceled`), `dragEnd` receives `{ canceled: true }` and the context **automatically restores the snapshot** — no consumer code needed.

### `NeoSortableContext` API

The context object passed to the `children` snippet exposes:

- `ctx.items` — `NeoSortableContextItems<Data>` — current items (reactive through the bound prop).
- `ctx.ids` — derived id list (array or record mirroring the `items` shape).
- `ctx.isDragging` — `boolean` — `true` while a drag is in flight.
- `ctx.get(id)` — `Sortable | undefined` — look up a registered sortable instance by id.
- `ctx.has(id)` — `boolean` — check whether an id is registered.
- `ctx.move(event)` — delegates to `@dnd-kit/helpers` `move`.
- `ctx.swap(event)` — delegates to `@dnd-kit/helpers` `swap`.

## Common pattern

### Single list

```svelte
<script lang="ts">
  import type { NeoSortableContextItems } from '@dvcol/neo-svelte/sortable';

  import { NeoSortableItem, NeoSortableProvider } from '@dvcol/neo-svelte/sortable';

  let items = $state<NeoSortableContextItems<{ label: string }>>([
    { id: '1', data: { label: 'Alpha' } },
    { id: '2', data: { label: 'Beta' } },
    { id: '3', data: { label: 'Gamma' } },
  ]);
</script>

<NeoSortableProvider bind:items axis="y">
  {#snippet children(ctx)}
    <ul>
      {#each ctx.items as item, index (item.id)}
        <NeoSortableItem {...item} {index}>
          {#snippet children({ instance, data })}
            <!-- apply {@attach instance.attach} to register the element with @dnd-kit -->
            <li {@attach instance.attach}>
              {data.label}
            </li>
          {/snippet}
        </NeoSortableItem>
      {/each}
    </ul>
  {/snippet}
</NeoSortableProvider>
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvcol/neo-svelte](https://github.com/dvcol/neo-svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
