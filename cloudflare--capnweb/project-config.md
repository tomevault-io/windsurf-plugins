---
trigger: always_on
description: Astro, with [Nimbus](https://nimbus-docs.com) (`@cloudflare/nimbus-docs`) as the docs framework. The
---

# The Cap'n Web docs site

Astro, with [Nimbus](https://nimbus-docs.com) (`@cloudflare/nimbus-docs`) as the docs framework. The
package handles content schemas, sidebar/TOC, MDX to markdown, search, OG cards, `llms.txt`, build
hooks, and the `nimbus-docs` CLI. Everything in `src/` is a real file in this repo and yours to
edit, including the files the scaffold wrote.

`README.md` next to this file explains why the site looks and works the way it does: the palette,
the page shell, the WebGL hero, the example playgrounds, the traps. Read it before changing anything
visual.

## Working in here

This package is part of the repo pnpm workspace. Install once at the repo root:

```sh
pnpm install
pnpm --filter capnweb-docs dev      # http://localhost:4321
pnpm --filter capnweb-docs build    # static output in ./dist
pnpm --filter capnweb-docs check    # astro check: types, content collections
pnpm --filter capnweb-docs lint:docs
```

The repo root `.npmrc` pins `@cloudflare` to the public registry so installs of
`@cloudflare/nimbus-docs` (and other public `@cloudflare` packages) work even when a user-level
npmrc maps that scope to an internal registry. Do not remove that line to "fix" a local registry
preference.

Production and PR preview deploys both go through Workers Builds
(`pnpm --filter capnweb-docs build` / `pnpm --filter capnweb-docs exec wrangler deploy`).
`README.md`, "Deployment" and "Previews", is the detail.

`predev` and `prebuild` build the library, then run `bundle-size` and `playgrounds`. The playground
bundler reads the library's **build output**, so that root build is required before demos show up.

## File layout

Where things are, and what each one is for:

```text
astro.config.ts              # nimbus(defineNimbusConfig({...})): sidebar, lint rules, markdown plugins
nimbus.json                  # what the scaffold and the registry installed. Committed.
.nimbus/                     # build scratch: materialized lint config, route manifest. Gitignored.
fonts/                       # build-time only, for the OG cards. Not under public/ on purpose.
scripts/
├── build-playgrounds.mjs    # bundles each example's worker + client into public/playground/
├── measure-bundle.mjs       # writes src/generated/bundle-size.json
└── mdast-bundle-size.mjs    # Sätteri plugin: %BUNDLE_SIZE% in .md bodies
src/
├── components.ts            # MDX globals registry -- every component used in .mdx must be listed
├── components/              # ours: Hero, Features, NavList, Playground, Prose, and
│                            #       canvas-hero/ (the landing figure and its harness)
│   └── ui/<slug>/           # from the Nimbus registry, plus AgentDirective, Header, Render
├── content/docs/**.{md,mdx} # the pages, one directory per sidebar group
├── content.config.ts        # docsCollection() + partialsCollection() + the %BUNDLE_SIZE% transform
├── examples.ts              # the single list of playground examples, read by pages and bundler
├── generated/               # bundle-size.json, written by prebuild. Gitignored.
├── layouts/                 # BaseLayout (head, theme bootstrap), DocsLayout (three columns)
├── lib/                     # cn.ts, source.ts (reads real files)
├── pages/                   # [...slug].astro, 404, llms.txt, robots.txt, og/
└── styles/                  # globals.css (tokens + shell), prose.css
public/                      # favicon, _headers, and the generated playground bundles
wrangler.jsonc               # static assets on a Worker, no script
```

## Writing docs

Frontmatter validates against Nimbus's `docsSchema`. `title` is required. Sidebar **groups** are
declared in `astro.config.ts`; position **within** a group comes from frontmatter:

```mdx
---
title: My page
description: One-line summary.
sidebar:
  order: 3
---

Content here. The H1 comes from `title` -- don't repeat it in the body.

## Section heading
```

Rules:

- **Components must be PascalCase and registered in `src/components.ts`.** A pre-build validator
  fails the build on an unregistered tag, with a "did you mean" hint.
- **Partials use `<Render file="..." />`.** Don't import `.mdx` directly.
- **Icons are `astro-icon` + Phosphor**: `<Icon name="ph:<glyph>" />`, imported from
  `@cloudflare/nimbus-docs/components/Icon.astro` rather than `astro-icon/components`, which is not
  a dependency here. Glyphs: [phosphoricons.com](https://phosphoricons.com).
- **A `mode: custom` page gets a bare `<main>`** -- no sidebar, no TOC, and no `.docs-content`
  wrapper or width cap either, so its prose must be wrapped in `<Prose>` or it renders unstyled and
  edge to edge.
- **Never type the library's size into prose.** Write `%BUNDLE_SIZE%` and it is substituted from the
  measured value, in bodies and in frontmatter alike.
- **Don't remove `<AgentDirective />` from `BaseLayout.astro`.** It points agents at `/llms.txt`.

House style for the prose itself: no em dashes (` -- ` in text, which the markdown pipeline leaves
alone), every code fence gets a language, and no code block directly under an `##` heading -- say
what it is first.

## Adding things


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/capnweb](https://github.com/cloudflare/capnweb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
