---
trigger: always_on
description: **Before writing any UI, runtime, `framework/uihost`, OR code that *emits*
---

# Claude / agent instructions for the GoFastr repo

**Before writing any UI, runtime, `framework/uihost`, OR code that *emits*
UI — the blueprint generator (`cmd/gofastr`), batteries, anything that
produces markup or CSS — read
[`core-ui/ARCHITECTURE.md`](core-ui/ARCHITECTURE.md). This is mandatory.
Emitting a styled `<div>` is writing UI. A generator that ships CSS is a
generator writing UI badly.**

**Before adding, moving, or extracting anything under `framework/`, read
[`framework/ARCHITECTURE.md`](framework/ARCHITECTURE.md).** It captures the
package layout, the layering rules, the cycle-breaking interfaces
(`entity.Registry`, `db.Executor`), and the recipe for new extractions.

The architecture document is the contract. It captures failure *classes*
caught the hard way so they don't repeat:
- **Navigation** — three wrong attempts before the SSR/island/SSE model
  was written down.
- **Styling/structure ownership** — caught 2026-06 building the Meridian
  flagship: the blueprint generator accreted ~70 bespoke CSS rules
  (`.mrd-*`, `.gofastr-*`, a `BlueprintBaseCSS` string) and hand-rolled
  markup that duplicated — or worked around — components that already
  existed (`ui.Grid`, `ui.Stack`, `ui.ThemeToggle`, the `--font-heading`
  token). The fix was to delete all of it and compose/extend the design
  system. The blueprint now ships **zero** CSS. See Hard rules 7–9.

**Before adding, renaming, or removing any exported API, route, CLI
subcommand, JSON field, or auto-generated artifact, the `gofastr-docs`
skill at `.claude/skills/gofastr-docs/SKILL.md` auto-loads. Docs ship
in the same commit as the code change — not a follow-up. The docs
live in `framework/docs/content/*.md` and are embedded into the
`gofastr` binary at build time — `gofastr docs` browses them; the
MCP tools `framework_docs_list` / `framework_docs_get` /
`framework_docs_search` expose them to agents connected to a live app.**

## TL;DR of the architecture (read the full doc anyway)

- **SSR-first**. Every page is fully server-rendered on initial load.
- **Hydration**, not re-render. `runtime.js` attaches handlers to the
  existing DOM after first paint.
- **Cross-page nav is client-side** with cache. No hard refreshes when
  going from `/a` to `/b`.
- **In-page state changes are islands**: a click fires an RPC, the
  server returns new island HTML, the runtime swaps just that island.
- **Server-pushed updates** flow through signals + SSE for genuine
  background events, not user actions.

## Hard rules

1. Never make in-page state changes (sort, paginate, expand) into routes.
   They are islands.
2. Never re-implement pagination/sort/filter math in JS. Server-side.
3. Never use SSE to deliver responses to user actions. SSE is push-only.
4. Never add `location.href = …` or full reloads as a "fix".
5. Never add new `data-fui-*` attributes without updating
   `core-ui/ARCHITECTURE.md` and the runtime test suite.
6. Never expose an entity holding per-user data via auto-CRUD without
   setting `EntityConfig.OwnerField`. See
   `framework/docs/content/entity-declarations.md` → "Per-user scoping".
7. **One styling surface.** Generators and apps ship ZERO bespoke CSS and
   ZERO hand-rolled structural markup. ALL styling + layout lives in the
   design system: `framework/ui` components (CSS via
   `registry.RegisterStyle`), `core-ui/app` layouts, `core-ui/style`
   tokens. A surface that needs styling the design system doesn't provide
   is a MISSING component / layout / token — add it *upstream* and compose
   it. Tripwires that mean STOP, you're drifting: a `*BaseCSS` string
   accreting rules; an invented `.mrd-*`/`.gofastr-*` class; setting a CSS
   property where a `var(--*)` token belongs; overriding a component's
   internals from outside instead of giving the component a config/variant.
8. **Survey before you build.** Before hand-rolling any UI markup or CSS,
   `grep` `framework/ui` + `core-ui` for an existing primitive — the
   catalog is large (Hero, Grid, Stack, Cluster, DetailList, Form,
   FormField, AuthCard, SiteHeader, Sidebar, ThemeToggle, Card, Section,
   DataTable, StatCard, charts, PageHeader, …). Reinventing or
   CSS-overriding an existing component is the #1 failure mode here. If
   it's genuinely missing, add it to the design system, not locally.
9. **Pixels, not probes — and dogfood the flagship.** Never claim a UI
   "works / polished / verified" from a DOM dump, a11y tree, or
   computed-style probe: those cannot see layout or composition.
   Screenshot the *rendered* page and read it (use `chromedp` →
   `FullScreenshot` to a PNG if Playwright hangs on the SSE connection).
   For any framework/design-system change, verify `examples/meridian`
   end-to-end — marketing, app, auth, admin, mobile, light + dark. Meridian
   is the design-system completeness canary: a surface there that needs
   CSS the components don't provide is a gap to fix upstream, never a patch.

## Common operations

- **Build / run the example website**: `./scripts/dev-watch.sh` (auto-rebuild + livereload, port `:8082`). Dev-watch writes to `/tmp/` because the watched tree must stay clean.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DonaldMurillo/gofastr](https://github.com/DonaldMurillo/gofastr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
