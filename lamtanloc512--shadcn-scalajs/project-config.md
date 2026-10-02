---
trigger: always_on
description: A port of shadcn/ui's philosophy to Scala.js + Laminar: components you copy into your own project (CLI + registry, like real shadcn/ui — not just a published library), styled with Tailwind CSS v4 utilities matching shadcn/ui's canonical `new-york-v4` source exactly (not basecoat CSS — see "History" below), and every component also compiles to a standalone Web Component so non-Scala frontends can use it too.
---

# shadcn-scalajs

## Project rules

### What this is

A port of shadcn/ui's philosophy to Scala.js + Laminar: components you copy into your own project (CLI + registry, like real shadcn/ui — not just a published library), styled with Tailwind CSS v4 utilities matching shadcn/ui's canonical `new-york-v4` source exactly (not basecoat CSS — see "History" below), and every component also compiles to a standalone Web Component so non-Scala frontends can use it too.

### History

v1 (5 components: Button, Badge, Dialog, Accordion, DropdownMenu) shipped styled with vendored, patched basecoat CSS loaded into each Web Component's Shadow Root. The project was then migrated wholesale to Tailwind CSS v4: components now carry shadcn/ui's own Tailwind utility classes directly (see any `packages/ui/*.scala` file — e.g. `Button.scala`'s `variantClasses`/`sizeClasses` maps are copied straight from `button.tsx`), and `apps/web` runs a real Tailwind v4 + PostCSS pipeline instead of linking a static CSS file. `vendor/basecoat-*.cdn.css` no longer exist — see `vendor/NOTICE.md` for exactly what's vendored now and why.

### Status

Component implementation status and the tier breakdown (pure Tailwind / native-element / hand-rolled state machine) now live in `packages/ui/CLAUDE.md` — read it before touching `packages/ui` or `packages/webcomponents`.

### Layout

```
apps/web/                 Vite site and registry host. sbt project id is still `site` (apps/web -> repo root is two levels, so vite `cwd` stays `"../.."`).
apps/docs/                Written guide (Astro Starlight). Not the component gallery.
packages/cli/             Published npm package: `init`, `add`, `mcp`.
packages/core/            CommonAttrs (openAttr), Tags (slot). Copied with components.
packages/ui/              Laminar component source of truth — what the CLI copies; one .scala + one .registry.json per component.
packages/blocks/          Page/section compositions built from packages/ui. Package-legal dir (`login01/`) plus a hyphenated `<name>.registry.json` sidecar.
packages/theme/           Style-pack CSS and tokens installed by `add`.
packages/webcomponents/   Experimental custom-element wrappers. Bundle built by apps/web/scripts/build-webcomponents.mjs.
vendor/                   shadcn/ui style-pack snapshots consumed by build-style-packs.mjs — see vendor/NOTICE.md.
docs/                     Contributor design notes. Not the published guide.
```

### Build/dev commands

```bash
# add coursier-installed sbt to PATH if `sbt` isn't found:
export PATH="$PATH:$HOME/Library/Application Support/Coursier/bin"

sbt core/compile ui/compile webcomponents/compile site/compile   # compile everything
sbt ui/fastLinkJS webcomponents/fastLinkJS site/fastLinkJS       # Scala.js link (dev / per module)
sbt siteOpt                                                      # size-optimized site/fullLinkJS (or `sbt opt` for ui+wc+site)
 sbt scalafmtAll                                                  # format before committing
sbt core/publishLocal                                            # publish core to ~/.ivy2/local (needed for consumer fixtures / real CLI testing)

cd apps/web && npm install && npm run dev   # predev runs build-style-packs + build-registry, then Vite
cd apps/web && npm run build                # production: Vite plugin runs site/fullLinkJS, then esbuild minify → dist/
# → http://localhost:4300/                    native Laminar landing page
# → http://localhost:4300/components          components index (componentsGalleryPage)
# → http://localhost:4300/components/<name>   per-component docs + live preview
# → http://localhost:4300/plain-html-demo.html  Web Component demo (zero Scala.js on the page)
cd apps/web && node scripts/build-registry.mjs   # regenerate public/registry/*.json from packages/ui (also runs as part of predev/prebuild)

cd packages/cli && npm install && npm run build  # -> dist/index.js
node packages/cli/dist/index.js init --registry <path-or-url> --source-dir <path>
node packages/cli/dist/index.js add <component...>

./scripts/test   # build + registry rebuild + CLI init/add smoke test against a temp dir
```

### Things that will bite you if you don't know them


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lamtanloc512/shadcn-scalajs](https://github.com/lamtanloc512/shadcn-scalajs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
