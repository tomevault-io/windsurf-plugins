---
trigger: always_on
description: > Agent-facing context. If you're picking up work on this site, start here.
---

# This Nimbus docs site

> Agent-facing context. If you're picking up work on this site, start here.

Astro-based docs site. The `nimbus-docs` package provides the integration, content schemas, navigation/sidebar/TOC computation, MDX→markdown rendering, build hooks, and the `nimbus` CLI. Everything you see in `src/` is user-owned and yours to edit.

## File layout

```
astro.config.ts                # imports `nimbus` and `defineNimbusConfig` — site config lives inline here
nimbus.config.ts               # (alternative) some projects split the Nimbus config into its own file
src/
├── components.ts              # MDX globals registry — every component used in .mdx files must be listed here
├── components/                # repo-owned components
│   ├── AgentDirective.astro   # ships an agent-readable hint into every doc page; do not remove
│   ├── Header.astro
│   ├── Render.astro           # partial loader — <Render file="..." />
│   └── ui/                    # registry-installed UI components (badge, dialog, sidebar, search, etc.)
├── content/
│   ├── docs/*.mdx             # docs content
│   └── partials/*.mdx         # partials referenced via <Render file="..." />
├── content.config.ts          # docsCollection() and partialsCollection() are registered here
├── layouts/
│   ├── BaseLayout.astro       # renders <NimbusHead /> + <AgentDirective />; wraps every page
│   └── DocsLayout.astro       # docs page chrome — sidebar, TOC, breadcrumbs, pagination
├── lib/
│   └── cn.ts                  # Tailwind className merger (clsx + tailwind-merge)
├── pages/
│   ├── index.astro            # landing
│   ├── [...slug].astro        # docs catch-all
│   ├── [...slug]/index.md.ts  # per-page markdown alternate (the .md sibling of every doc URL)
│   ├── llms.txt.ts            # /llms.txt
│   ├── og.png.ts              # site-level OG image
│   ├── og/
│   │   ├── _renderer.ts       # shared OG card renderer (underscore = not a route)
│   │   └── [...slug].ts       # per-page OG image
│   └── robots.txt.ts          # /robots.txt
└── styles/
    ├── globals.css
    └── prose.css
```

For Cloudflare deploys, also: `wrangler.jsonc` at project root.

## Writing docs

Frontmatter must validate against `docsSchema` from `nimbus-docs/schemas`. Required: `title`. The schema includes optional fields for description, sidebar overrides, drafts, dates, edit-link suppression — read the schema for the full shape.

```mdx
---
title: My page
description: One-line summary.
---

# My page

Content here.
```

**MDX components must be PascalCase and registered.** Every component used in a `.mdx` file (`<Steps>`, `<Card>`, etc.) must appear in `src/components.ts`. A pre-build validator catches typos and unregistered components with `file:line:column` and a "did you mean" hint.

**Partials use `<Render />`.** Don't import `.mdx` files directly. Put shared content in `src/content/partials/<slug>.mdx`, then reference with `<Render file="<slug>" />`. The `Render` component emits a "did you mean" diagnostic for unknown slugs.

**Icons render via `astro-icon` + Phosphor.** Use `<Icon name="ph:<glyph>" class="w-4 h-4" />` from `astro-icon/components`. Don't reintroduce inline `<svg>` blocks for icons. Browse glyphs at [phosphoricons.com](https://phosphoricons.com).

**`AgentDirective` renders in `BaseLayout.astro`.** It writes an agent-readable hint at the top of every doc and markdown alternate pointing at `/llms.txt`. Don't remove it.

## Adding things

| Goal | Action |
|---|---|
| New doc page | Create `src/content/docs/<slug>.mdx` with valid frontmatter. The sidebar picks it up automatically. |
| New partial | Create `src/content/partials/<slug>.mdx`. Use via `<Render file="<slug>" />`. |
| New UI component from the registry | `pnpm exec nimbus-docs add <slug>`. Resolves dependencies and writes files into `src/components/ui/<slug>/`. Remember to import + register the component in `src/components.ts` if it's used in MDX. |
| New feature (e.g. custom 404, AI surface) | `pnpm exec nimbus-docs add <feature-slug>`. Prints an agent brief; pipe it to your coding agent. |
| New custom page route | Add a file under `src/pages/`. |
| Custom OG card style | Edit `src/pages/og/_renderer.ts`. |

List installable items with `pnpm exec nimbus-docs list`.

## Audit this site

When asked to check or audit the site, walk the categories below. For each finding, emit a bullet:

```
- [error|warn|info] FILE:LINE — what's wrong + why it matters + recommended fix.
```

End the report with: `Summary: N errors, N warnings.`

### Config
- `astro.config.ts` imports `nimbus` and calls it with the result of `defineNimbusConfig({ ... })`.
- `site` is a non-empty URL. Watch for trailing-slash mismatches against page URLs.
- `editPattern` (if set) includes the literal `{path}` placeholder.
- Every sidebar reference resolves to a real content entry.
- Astro `output:` matches the deploy target (`static` for static deploys).

### Content collections
- `src/content.config.ts` registers `docsCollection()` from `nimbus-docs/content`. Register `partialsCollection()` too if `src/content/partials/` exists.
- Every `.mdx` file lives inside a registered collection. Loose `.mdx` under `src/content/` outside a registered collection won't be picked up.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/nimbus](https://github.com/cloudflare/nimbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
