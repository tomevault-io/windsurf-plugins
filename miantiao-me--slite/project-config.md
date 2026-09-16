---
trigger: always_on
description: - Slite and [Sink](https://github.com/miantiao-me/Sink) are sibling versions of the same link-management and analytics project. Sink is not legacy or deprecated, and Slite is not a fork replacement for it.
---

# Slite Repository Guide

## Sibling versions

- Slite and [Sink](https://github.com/miantiao-me/Sink) are sibling versions of the same link-management and analytics project. Sink is not legacy or deprecated, and Slite is not a fork replacement for it.
- Keep features, API contracts, and file organization compatible with Sink wherever practical.
- The intended differences are limited to runtime and storage: Slite removes the Cloudflare runtime and runs as a local Node.js/Docker process with SQLite authoritative storage, an unstorage memory link cache, unstorage filesystem images and backups, and DuckDB analytics.
- When porting a feature from Sink, compare against the Sink implementation first and avoid unnecessary renames or contract drift.

## Non-obvious constraints

- Write all documentation and code comments in English.
- Use Node.js 24 or newer and pnpm 11.11.0 (`package.json` is authoritative). The root package is the Nuxt app; `docs/` is the `@slite/docs` VitePress workspace package.
- Do not hand-edit `app/components/ui/**`; it is managed by shadcn-vue and excluded from ESLint.
- Read `DESIGN.md` before UI work. The authoritative design sources are `app/assets/css/tailwind.css` and `app/components/ui/**`; `DESIGN.md` is a derived summary.
- Do not invent undocumented design tokens.
- Feature-level classes should stay focused on layout and composition. Prefer shared component variants and sizes over overriding primitive chrome such as radius, border, shadow, background, typography, padding, height, or focus, hover, and disabled states. Recurring product-specific exceptions should become app-owned wrappers outside `app/components/ui/**`.
- Use `DropdownMenu` for compact contextual action lists and `Popover` for richer anchored content; do not simulate menu items with buttons inside a `Popover`.
- Compose dashboard navigation and utilities with `SidebarMenu`, `SidebarMenuItem`, and `SidebarMenuButton`; do not recreate sidebar hover, focus, radius, or collapsed behavior with raw controls.
- After changing design tokens or `DESIGN.md`, run `npx @google/design.md lint DESIGN.md` and resolve all errors.
- Nuxt and server utilities are auto-imported. Follow nearby code before adding explicit imports for framework globals.
- Use `@lucide/vue` for Lucide icons; do not add `lucide-vue-next`.
- Application forms must live in dedicated `*Form.vue` components and should prefer `@tanstack/vue-form`. Generated components under `app/components/ui/form/**` may use `vee-validate` internally.
- Business dialogs must live in dedicated `*Dialog.vue` or `*Modal.vue` components; use `AlertDialog` for confirmations and `ResponsiveModal` for task content that adapts between dialog and drawer, and do not inline these implementations in unrelated components.
- Locale messages live in `i18n/locales/<locale>/*.json` and are loaded through the module list in `i18n/i18n.ts`. Organize feature messages by their owning product domain; do not introduce cross-cutting `ux`, `ui`, or `messages` namespaces at the top level or across product domains.
- Keep every locale directory aligned on module files, translation keys, and interpolation placeholders. When moving a key or changing the module list, update every locale and all application references in the same change.

## Setup and commands

```bash
pnpm install                              # also runs build:map, nuxt prepare, and hook setup
pnpm dev                                  # Nuxt dev server on port 5483
pnpm build                                # production build with an 8 GB Node heap
pnpm preview                              # requires existing .output build artifacts
pnpm dev:docs                             # VitePress docs dev server
pnpm build:docs                           # production docs build
pnpm preview:docs                         # preview the docs build
pnpm lint                                 # check only
pnpm lint:fix                             # modifies files
pnpm types:check
pnpm test --run                           # full Vitest run, not watch mode
pnpm test --run tests/api/link.spec.ts    # one test file
pnpm test --run -t 'creates new link'     # tests matching a name
```

- ESLint and TypeScript extend generated `.nuxt` files. If they are missing, run `pnpm postinstall` (or `pnpm install`) before diagnosing config errors.
- Tests that authenticate against a running server configure their own `NUXT_SITE_TOKEN`; local values are loaded from `.env`, with `.env.example` as the template.
- There is no validation CI workflow. Run the relevant lint, typecheck, and test commands locally.
- The pre-commit hook only runs `eslint --fix` on staged JS/TS/Vue files; it does not replace full-project verification.

## Architecture and data flow

- `app/` is a client-only Nuxt 4 UI (`ssr: false`), and `/dashboard` redirects to `/dashboard/links`. `server/` is the Nitro Node.js backend. Run one process per local persistent data directory; no clustered workers, shared network volumes, or replicas sharing data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miantiao-me/Slite](https://github.com/miantiao-me/Slite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
