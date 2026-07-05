---
trigger: always_on
description: This is a pnpm + Turborepo monorepo. Multiple Next.js apps share code via `@cant/shared`.
---

# Claude Code Rules

## Monorepo structure

This is a pnpm + Turborepo monorepo. Multiple Next.js apps share code via `@cant/shared`.

```
apps/cant-maintain    # React component API patterns
apps/cant-resize      # Responsive design patterns
apps/cant-type        # TypeScript patterns
apps/cant-orchestrate # Container orchestration patterns
apps/cant-seo         # SEO best practices for Next.js
apps/cant-ux          # UX design patterns
apps/cant-explode     # Chemistry patterns (molecules, reactions, structures)
apps/cant-branch      # Git version control patterns
apps/cant-query       # API design patterns
apps/cant-test        # Testing patterns (+ Bug Hunt game)
apps/cant-game        # Game development patterns
apps/cant-ticket      # Agile ticket craft and estimation
apps/cant-hub         # Series hub / landing page / screening
packages/shared       # @cant/shared - components, game logic, challenges, utilities
```

## Before committing

Run checks from the repo root:

```bash
pnpm check
```

This runs all four checks together (lint, typecheck, test, format:check) via Turborepo, with `--continue` so every failure is reported in one pass. To run them individually:

```bash
pnpm lint
pnpm typecheck
pnpm test
pnpm format:check
```

If formatting fails, run `pnpm format` from the repo root (or `npx prettier --write .` in a single app directory).

Do not commit code that fails any of these checks.

`pnpm test` runs each app's vitest suite via Turborepo (apps without a `test` script are skipped). Add tests as colocated `lib/**/*.test.ts` files; see `apps/cant-ticket` or `apps/cant-test` for the setup.

If you encounter pre-existing lint or type errors in files you did not change, fix them immediately. Do not ignore them or defer them as "pre-existing." The codebase must be clean after every session.

When your changes affect project structure (adding/removing/moving apps, files, directories, exports, or dependencies), update all documentation that references the old structure before committing. Check at minimum:

- `CLAUDE.md` (this file) — monorepo structure, shared exports, adding apps/challenges
- `README.md` — app table, project structure tree, "what stays per-app," adding a new app
- App-specific `CLAUDE.md` files — challenge paths, app-specific notes

To check a single app: `pnpm turbo lint --filter=cant-maintain`

## Working with Turborepo

- Use filtered commands when working on one app: `pnpm dev:maintain`, `pnpm build:resize`, `pnpm dev:seo`
- `pnpm dev` starts all apps simultaneously (resource-heavy, avoid unless needed)
- Turbo caches builds. If you change shared code, dependent apps rebuild automatically
- The `build` task depends on `^build` (shared package builds first)

## Working with @cant/shared

### When to add to shared

- Component exists in 2+ apps with identical or near-identical code
- Utility function is used across apps
- **Every shared component must have a Storybook story.** When adding or modifying a component in `packages/shared/src/components/`, create or update its story in the `__stories__/` directory next to it. This is not optional.

### When to keep per-app

- Component uses app-specific challenge data or categories
- File has fewer than ~3 lines of shared logic (re-export is fine, don't over-abstract)
- Theme colors, landing pages, app-specific features (viewer, playground, inspector, changelog)

### Pattern: thin wrappers

Apps import shared components and pass app-specific config as props:

```tsx
// apps/cant-resize/components/site-footer.tsx
import { SiteFooter as SharedSiteFooter } from "@cant/shared/components/site-footer";

const NAV_LINKS = [
  { href: "/canvas", label: "Viewer" },
  { href: "/play", label: "Play" },
  { href: "/learn", label: "Learn" },
  {
    href: "https://github.com/saschb2b/cant-resize",
    label: "GitHub",
    external: true,
  },
];

export function SiteFooter() {
  return <SharedSiteFooter navLinks={NAV_LINKS} />;
}
```

### Adding shared exports

When adding new files to `packages/shared/src/`, check that the export pattern in `packages/shared/package.json` covers it. Current patterns:

- `./components/*` maps to `./src/components/*.tsx`
- `./components/game/*` maps to `./src/components/game/*.tsx`
- `./lib/*` maps to `./src/lib/*.ts`
- `./lib/game/*` maps to `./src/lib/game/*.ts`
- `./lib/challenges/*` maps to `./src/lib/challenges/*/index.ts`

Each app's `next.config.mjs` includes `transpilePackages: ["@cant/shared"]`.

### App registry

All apps are registered in `packages/shared/src/lib/cant-apps.ts`. Each entry includes the app name, description, theme colors, icon SVG content, and cross-promo text. Update this file when adding a new app.

The `CantSeriesGrid` component (`packages/shared/src/components/cant-series-grid.tsx`) renders the cross-links section on landing pages (`variant="full"`) and play lobbies (`variant="compact"`). It reads from the app registry.

## Working with Storybook

Run: `pnpm storybook` (opens on :6006)

### Adding a story

Create a `.stories.tsx` file next to the component in a `__stories__` directory:

```
packages/shared/src/components/__stories__/my-component.stories.tsx
packages/shared/src/components/game/__stories__/my-game-component.stories.tsx
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saschb2b/cant](https://github.com/saschb2b/cant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
