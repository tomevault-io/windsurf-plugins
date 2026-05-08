---
trigger: always_on
description: You are an expert SvelteKit engineer. Follow these rules ALWAYS when generating or editing code.
---


# Cursor Project Rules — SvelteKit (latest) + Runes + TS + Supabase Auth + Postgres + shadcn-svelte

You are an expert SvelteKit engineer. Follow these rules ALWAYS when generating or editing code.

============================================================
0) Core principles
============================================================
- Prefer clarity + consistency over cleverness.
- Make changes minimal and localized.
- Never introduce new dependencies unless explicitly requested. If a dependency is truly necessary, explain why and propose alternatives.
- Preserve existing patterns in the codebase (naming, folder structure, error handling, UI primitives).
- Avoid breaking public APIs/exports without updating all usage sites.
- Default to shadcn-svelte components for UI primitives unless a plain semantic element is clearly better.
- MCP usage (required):
  - Use the Supabase MCP for database/auth inspection and project context in READ-ONLY mode by default.
  - Do not run mutating Supabase MCP operations (writes, deletes, schema changes) unless explicitly requested by the user.
  - Use the Svelte MCP whenever Svelte/SvelteKit development is involved.
  - For Svelte MCP workflows: run `list-sections` first and then fetch ALL relevant sections via `get-documentation` after reviewing `use_cases`.
  - Use `svelte-autofixer` whenever writing/editing Svelte code, and keep running it until no issues/suggestions remain.
  - Use `playground-link` only when the user explicitly asks for a playground link, and never when code is being written directly to project files.

============================================================
1) SvelteKit conventions
============================================================
- Use SvelteKit file conventions correctly:
  - Route components: +page.svelte, layouts: +layout.svelte, data: +page.ts/+layout.ts.
  - Avoid putting business logic in route files; move reusable logic to src/lib.
- In SPA mode (ssr=false), do not rely on server-only behavior. Keep Supabase client-side with browser-safe env vars only.
- Use $app/navigation (goto) and $app/state (page, navigating, updated) for routing concerns.
  - NEVER use deprecated $app/stores - it's for legacy SvelteKit <2.12 only.
  - Import page from $app/state: `import { page } from '$app/state'`
  - Use `page.url.pathname` directly (no $ prefix needed - it's already reactive with runes).
- Redirects/guards must be done in layouts for sections (e.g., protected area), not sprinkled across many pages.
- Never compare pathnames with raw string equality — SvelteKit may add or omit trailing slashes.
  - Always normalize paths before comparing (strip trailing slash if length > 1).
  - Prefer comparing parsed identifiers (e.g., step numbers, route params) over raw pathname strings.
- Load function invalidation with `depends`/`invalidate`:
  - When a `+layout.ts` or `+page.ts` load function depends on auth state, call `depends('auth:session')` inside it so it can be re-run on demand.
  - After login, logout, or any auth state change, call `invalidate('auth:session')` (from `$app/navigation`) to trigger re-runs of every load function that declared the dependency.
  - This keeps load functions reactive to auth changes without sprinkling manual checks across pages.
  - The dependency key is a custom string — not a URL. SvelteKit matches it by exact string equality.

============================================================
2) Runes + state management
============================================================
- Use Runes ($state, $derived, $effect) and avoid legacy writable stores.
- $derived must be side-effect free:
  - Never call `goto()`, network requests, or mutate $state inside $derived.
  - $derived is for pure computations only — use $effect for side effects that react to derived values.
  - Example: derive a `redirecting` flag with `$derived(status === 'needs-auth')`, then perform `goto()` in a separate `$effect` that reads `status`.
- Avoid module-level side effects that register listeners on import (especially auth listeners). Use an explicit init function called once (e.g., from root layout onMount) or idempotent subscription guards.
- Keep state minimal:
  - Put global app/auth state in src/lib (e.g., src/lib/auth/state.svelte.ts).
  - Keep page-local state inside the page component unless it must be shared.
- $effect must be cleanup-safe:
  - Always return cleanup functions when subscribing to listeners.
  - Prevent duplicate subscriptions in dev/HMR.
- $effect dependency tracking is synchronous only:
  - Svelte 5 only tracks reactive reads that happen synchronously in the $effect body.
  - Any reactive value read inside an async callback (.then(), await, setTimeout) is NOT tracked.
  - If an $effect needs to re-run when a reactive value changes, read it into a local variable at the TOP of the effect body, before any async code.
  - Example: `$effect(() => { const path = page.url.pathname; void fetchData().then(() => { /* use path here */ }); });`
- Layout guards that gate children rendering must account for all valid sub-states:
  - If a layout conditionally renders `{@render children()}`, ensure every legitimate child route has a state that passes the gate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesselgrift/sveltekit-spa](https://github.com/wesselgrift/sveltekit-spa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
