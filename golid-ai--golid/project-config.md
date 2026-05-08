---
trigger: always_on
description: Patterns for SolidJS route pages — data fetching, state, modals, auth
---


# SolidJS Page Patterns

Follow the established patterns in `dashboard/index.tsx` and `settings/index.tsx`.

## Data Fetching

**Always use `onMount` + `createSignal` + `alive` guard + `batch`.** This is the established pattern across all pages. Do not use `createResource` — it causes orphaned computation warnings on route transitions and triggers route-level Suspense when inside conditional components.

> `createAsync` + `query` from `@solidjs/router` is the official SolidJS recommendation going forward (Solid 2.0). Not yet adopted in this project — use the signals pattern for consistency.

```tsx
// STANDARD — used on all pages
const [data, setData] = createSignal<MyType[]>([]);
const [loading, setLoading] = createSignal(true);
const [error, setError] = createSignal("");

let alive = true;
onCleanup(() => {
  alive = false;
});

const fetchData = async () => {
  setLoading(true);
  setError("");
  try {
    const result = await myApi.list();
    if (!alive) return;
    batch(() => {
      setData(result);
      setLoading(false);
    });
  } catch (err) {
    if (!alive) return;
    batch(() => {
      setError(getErrorMessage(err, "Failed to load"));
      setLoading(false);
    });
  }
};

onMount(() => {
  fetchData();
});
```

## Async Data Pattern (batch + defer + onMount)

Three rules that MUST be used together for any page that fetches data:

**1. `batch()` signal updates** — wrap `setData` + `setLoading(false)` in `batch()` so they update atomically. Without this, nested reactive scopes resolve in intermediate states during route transitions, creating computations outside the reactive root.

**2. `onMount` for initial fetch** — never rely on `createEffect` for the first fetch.

**3. `defer: true` on `createEffect(on(...))` for reactive refetches** — prevents the effect from firing synchronously during mount (which overlaps with route transitions).

**Exception: Auth guards** use `on()` without `defer` — they must fire synchronously to prevent flash of unauthorized content. SSR middleware handles the server-side redirect; the client-side effect is a safety net for SPA navigation. See `routes/(private).tsx`.

```tsx
import { batch, onMount, createEffect, on } from "solid-js";

const fetchData = async () => {
  setLoading(true);
  setError("");
  try {
    const result = await api.list({ page: page() });
    if (!alive) return;
    batch(() => {
      // ← atomic update
      setData(result);
      setLoading(false);
    });
  } catch (err) {
    if (!alive) return;
    batch(() => {
      setError(getErrorMessage(err, "Failed to load"));
      setLoading(false);
    });
  }
};

onMount(() => {
  fetchData();
}); // ← initial fetch
createEffect(
  on(
    () => [filter(), page()] as const,
    () => {
      fetchData();
    },
    { defer: true }, // ← skip synchronous initial run
  ),
);
```

## Alive Guard

Every component with async operations needs the cleanup guard:

```tsx
let alive = true;
onCleanup(() => {
  alive = false;
});
```

Check `if (!alive) return;` before every signal setter after an `await`.

````

## Signal-Driven Modals

Detail views within list pages use a signal, not a sub-route:

```tsx
const [activeItem, setActiveItem] = createSignal<Item | null>(null);
// Open: setActiveItem(item)
// Close: setActiveItem(null)

<Show when={activeItem()}>
  {(item) => <DetailModal item={item()} onClose={() => setActiveItem(null)} />}
</Show>
````

## Destructive Actions

Never `window.confirm()`. Always use `DestructiveModal`:

```tsx
const [deleteTarget, setDeleteTarget] = createSignal(false);
// ...
<DestructiveModal
  open={deleteTarget()}
  onOpenChange={(open) => {
    if (!open) setDeleteTarget(false);
  }}
  onConfirm={handleDelete}
  title="Delete item?"
  message="This action cannot be undone."
  confirmText="Delete"
/>;
```

## Page Titles (NEVER use reactive expressions)

**NEVER put reactive expressions inside `<Title>`.** This includes `createMemo` — any reactive getter call inside `<Title>` children creates a computation that leaks during route transitions, causing "computations created outside createRoot" warnings.

```tsx
// BAD — reactive ternary leaks during route transition
<Title>{userType() === "admin" ? "Manage Items" : "Items"} | My App</Title>

// ALSO BAD — createMemo is still reactive, still leaks
const pageTitle = createMemo(() => "Items");
<Title>{pageTitle()} | My App</Title>

// GOOD — static string, no reactive expression
<Title>Items | My App</Title>
```

## SSR Safety (CRITICAL)

SolidStart renders components on the server during SSR. Any access to browser APIs crashes the production build and serves blank pages.

- **Guard browser APIs** with `if (typeof window !== "undefined")` or use them only inside `onMount`.
- **Heavy browser-only components** (Three.js, WebRTC, charts, video) must be `lazy()` + `<Suspense>`.
- **Never import browser-only libraries at module top level** — use dynamic `import()` inside `onMount`.
- **`createEffect` runs on server** — don't access `window`/`document` in effects without a guard.

```tsx
// BAD — crashes SSR
const width = window.innerWidth;

// GOOD — only runs in browser
onMount(() => {
  const width = window.innerWidth;
});

// GOOD — lazy-loaded browser component

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [golid-ai/golid](https://github.com/golid-ai/golid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
