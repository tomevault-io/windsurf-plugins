---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# The Pragmatic Architect Specification

## 1. Core Persona and Operational Philosophy

<core_principles>
* **OOP & Encapsulation:** Utilize strict Object-Oriented Programming practices and encapsulation WHEREVER appropriate. Isolate data states and expose only the strictly necessary interfaces. Protect internal module states from global mutation.
* **DRY (Don't Repeat Yourself):** Abstract repetitive logic into unified, authoritative modules, hooks, or utility classes.
* **Orthogonality:** Keep components fundamentally independent. Changes in one domain must not cause cascading side effects in another.
* **Tracer Bullets:** Build small, end-to-end functional increments to prove an architectural concept before bulk-generating features.
* **Eradicate AI Slop:** Never output massive, tangled code dumps. Never recreate components that already exist in the repository. Stop and think before you generate. ALWAYS output your reasoning steps inside a `<thinking>` XML block before outputting executable code.
</core_principles>

## 2. Architectural Guardrails and File Constraints
*(Add specific file size limits, line counts, or directory structures here if needed)*

## 3. Frontend State Management Architecture

<state_management>
You must strictly and flawlessly separate server state from client state. NEVER conflate the two.

**Server State (TanStack Query):**
* Use TanStack Query (formerly React Query) for ALL asynchronous data fetching, caching, API synchronization, background updates, and server mutations.
* NEVER use `useEffect` combined with `useState` for data fetching or loading indicators. This is an anti-pattern.
* Encapsulate all queries and mutations within custom hooks (e.g., `useUserProfileQuery()`). Components must only consume the `data`, `isLoading`, and `error` objects.

**Client State (Zustand):**
* Use Zustand for ALL synchronous, ephemeral, and UI-driven global state (e.g., modal visibility, theme switching, multi-step form progression).
* Keep Zustand stores highly granular and modular. Do not create a single monolithic store.
* **Critical:** Always use selectors to extract state from Zustand stores to prevent unnecessary component re-renders (e.g., `const isOpen = useModalStore((state) => state.isOpen)`).
</state_management>

## 4. Animation and Performance Protocol (transitions.dev)

<animation_protocol>
Animations use **transitions.dev** — portable CSS transitions namespaced under `t-*` with semantic custom properties. CSS-first (no JS motion library). Browser performance and 60fps fluidity are non-negotiable.

* **Single source of tokens:** All durations/eases/distances live in the `:root` block in `src/app/transitions.css`. Snippets read those semantic names (`--modal-*`, `--dropdown-*`, `--panel-*`, `--shake-*`, …); never hardcode timings inline.
* **Use the catalog:** Reach for the matching transition — modal, menu-dropdown, panel-reveal, page-enter, icon-swap, text-swap, number-pop-in, success-check, error-state-shake, etc. Don't invent ad-hoc keyframes when one fits.
* **Prevent Layout Thrashing:** Only animate composite properties (`transform`, `opacity`, `filter`). NEVER animate layout properties (`width`, `height`, `top/left`, `margin`, `padding`).
* **Hardware Acceleration:** Apply `will-change: transform` to elements under heavy/continuous animation. Not global.
* **Reduced motion:** Every animation MUST keep its `@media (prefers-reduced-motion: reduce)` guard.
* **Radix/shadcn:** Components driven by Radix (`data-state`) animate via keyframes (so Radix Presence's `animationend` fires before unmount) — retuned to the transitions.dev easing/scale tokens, not the snippet's `.is-closing` JS orchestration.
* **Replay pattern:** To replay an animation (shake, pop-in), remove the class → force reflow (`void el.offsetWidth`) → re-add. Use the `useShake`-style hook, not raw GSAP.
* GSAP has been removed. Smooth scrolling still uses Lenis (§4.1) — that is scroll, not element animation.
</animation_protocol>

### 4.1 Mandatory UX Conventions (NON-NEGOTIABLE)

<ux_conventions>
These apply to every component and route. Do not ship UI that violates them.

* **Skeleton loaders for genuine loads:** A real loading state (no data yet) MUST render a skeleton placeholder (shadcn `Skeleton`), never a spinner or blank screen. The skeleton must mirror the final layout's shape.
* **Skeletons are cache-aware:** Only show a skeleton when there is genuinely no data. For client-fetched data, drive it off the query's `isLoading` (which is `false` when TanStack Query serves from cache) — never show a skeleton over cached content. Do NOT add a route-level `loading.tsx` for client-fetched pages: the server Suspense fallback can't see the client cache and flashes a skeleton on every navigation, defeating the cache. Reserve `loading.tsx` for pages whose data is fetched on the server.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redstone-md/g3](https://github.com/redstone-md/g3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
