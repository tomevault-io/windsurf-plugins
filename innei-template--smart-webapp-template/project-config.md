---
trigger: always_on
description: Guidance for AI code agents working in this repository. This document codifies project-specific rules, patterns, and safe operations. When in doubt or on conflict, prefer the actual codebase.
---

# agents.md

Guidance for AI code agents working in this repository. This document codifies project-specific rules, patterns, and safe operations. When in doubt or on conflict, prefer the actual codebase.

Scope:

- Stack: Vite + React 19 + TypeScript, TailwindCSS v4, Radix UI, Jotai, TanStack Query, Framer Motion (LazyMotion), React Router with vite-plugin-route-builder.
- Package manager: pnpm (required). See package.json for script names and versions.

Core commands:

- Development: pnpm dev
- Build: pnpm build
- Preview: pnpm serve
- Lint: pnpm lint
- Format: pnpm format

Repository rules (must follow):

- Never edit auto-generated files (e.g., src/generated-routes.ts). Add/rename files under src/pages/ to affect routing.
- Use the path alias ~/ for all src imports (configured in tsconfig).
- Use Framer Motion’s LazyMotion with m._ components only. Do not use motion._ directly.
- Prefer Spring presets from ~/lib/spring for animations.
- Use the Pastel color system classes instead of raw Tailwind colors.
- Follow component organization:
  - Base UI primitives -> src/components/ui/
  - App-shared (non-domain) -> src/components/common/
  - Feature/domain modules -> src/modules/<domain>/
- State management via Jotai with helpers from ~/lib/jotai. Atoms live in src/atoms/.
- Do not rely on the global location object. Use the stable router utilities (~/atoms/route) or React Router hooks through the StableRouterProvider.
- Keep JSX self-closing where applicable; adhere to eslint-config-hyoban and Prettier settings.

Routing and layouts:

- File-based routing via vite-plugin-route-builder.
  - Sync routes: \*.sync.tsx (no code-splitting)
  - Async routes: \*.tsx (lazy loaded)
  - Layout files: layout.tsx within a segment; render children via <Outlet />
- Example segment structure (do not edit src/generated-routes.ts directly):
  - src/pages/(main)/index.sync.tsx -> root route
  - src/pages/(main)/about.sync.tsx -> /about
  - src/pages/(main)/settings/layout.tsx -> wraps /settings subtree

Providers:

- Root providers are composed in src/providers/root-providers.tsx and include:
  - LazyMotion + MotionConfig
  - TanStack QueryClientProvider
  - Jotai Provider with a global store
  - Event, Context menu, and settings sync providers
  - StableRouterProvider to stabilize routing data and navigation
  - ModalContainer and Toaster
- Add new cross-cutting providers here, keeping order and side effects in mind.

Animation rules:

- Always use m.\* components imported from motion/react.
- Prefer transitions from Spring presets for consistency and bundle efficiency.

Example (animation):

```
import { m } from 'motion/react'
import { Spring } from '~/lib/spring'

export function AnimatedCard(props: { children?: React.ReactNode }) {
  return (
    <m.div
      initial={{ opacity: 0, y: 8 }}
      animate={{ opacity: 1, y: 0 }}
      transition={Spring.presets.smooth}
      className="rounded-lg bg-fill shadow-sm p-4"
    >
      {props.children}
    </m.div>
  )
}
```

Jotai patterns:

- Use createAtomHooks and createAtomAccessor from ~/lib/jotai for consistent access, hooks, and selectors.
- Keep atoms in src/atoms/; co-locate selectors next to atoms when domain-specific.

Example (atom + hooks):

```
import { atom } from 'jotai'
import { createAtomHooks, createAtomAccessor } from '~/lib/jotai'

const baseCounterAtom = atom(0)

// Typed hooks: [atomRef, useAtomHook, useValue, useSetter, get, set]
export const [
  counterAtom,
  useCounter,
  useCounterValue,
  useSetCounter,
  getCounter,
  setCounter,
] = createAtomHooks(baseCounterAtom)

// Optional: selectors
export const useIsEven = () => {
  const value = useCounterValue() // read-only value hook
  return value % 2 === 0
}
```

Stable routing patterns:

- Read-only route data and stable navigate are provided via ~/atoms/route and set by src/providers/stable-router-provider.tsx.
- Prefer useReadonlyRouteSelector for reading route state without causing re-renders.
- Prefer getStableRouterNavigate for imperative navigation outside React components.

Example (route utilities):

```
import { useReadonlyRouteSelector, getStableRouterNavigate } from '~/atoms/route'

export function RouteAwareComponent() {
  const pathname = useReadonlyRouteSelector((r) => r.location.pathname)
  const params = useReadonlyRouteSelector((r) => r.params)
  const navigate = getStableRouterNavigate()

  const goHome = () => navigate('/', { replace: true })

  return (
    <div className="text-text">
      <div>Pathname: {pathname}</div>
      <div>Params JSON: {JSON.stringify(params)}</div>
      <button className="btn" onClick={goHome}>Go Home</button>
    </div>
  )
}
```

UI components:

- Prefer primitives in src/components/ui/ for buttons, inputs, select, switch, slider, dialogs, context menus, etc.
- Compose primitives for feature-level components under src/modules/<domain>/.
- Use the Pastel color tokens (e.g., text-text, bg-background, border-border, bg-fill, bg-accent).

Example (simple page using primitives):

```
import { Button } from '~/components/ui/button/Button'
import { Divider } from '~/components/ui/divider/Divider'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [innei-template/smart-webapp-template](https://github.com/innei-template/smart-webapp-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
