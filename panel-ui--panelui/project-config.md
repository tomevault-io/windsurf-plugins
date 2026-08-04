---
trigger: always_on
description: High-performance React Native UI library for Expo, published on npm as
---

# PanelUI

High-performance React Native UI library for Expo, published on npm as
[`panelui-native`](https://www.npmjs.com/package/panelui-native).
GitHub: https://github.com/panel-ui/PanelUI

## Research before you build

**Never design a component, variant, animation, or token from scratch.** Before writing any
component code, read how the problem is already solved by mature libraries, then adapt it to
PanelUI's tokens and conventions. This is not optional.

Where to look, in order:

1. The React Native / Expo component libraries — for native structure, Reanimated usage, gesture
   handling, accessibility props and compound anatomy. Closest to our target; check here first.
2. The web component libraries — for compound-component API shape, prop naming and variant
   taxonomy. Their structure ports; their CSS does not.
3. The design-system references — for token usage and visual language.

The `.claude/skills/` directory holds the pinned references for 2 and 3; invoke them by name
before touching design tokens, `packages/panelui/theme.css`, the docs theming or the landing
page. They carry rules worth not reconstructing — for instance, *never rewrite `--alpha()` to
`rgba()` in the web CSS*: it is a valid Tailwind v4 build-time function, not broken CSS.

If none of them has the component, search the web for other React Native / Tailwind
implementations before inventing an approach.

How to read a repository:

- Use `gh api "repos/<owner>/<repo>/git/trees/main?recursive=1" --jq '.tree[].path'` to locate
  files, then `gh api "repos/<owner>/<repo>/contents/<path>" --jq '.content' | base64 -d` to read
  them. **Prefer `gh` over WebFetch** — `raw.githubusercontent.com` returns 404 for many repos.
- Some libraries ship a `<name>.md` next to each component with the full documented API. Read it
  before the implementation; it is faster and more accurate.

### Never name a reference library in anything we ship or author

This is a hard rule, and it applies to **source comments, JSDoc, README files, docs pages, npm
metadata and commit messages** alike:

- No `Adapted from: <repo>` headers. Write a header comment that explains what the component
  does and *why it is shaped that way* — that is the part worth keeping, and it stays true when
  the upstream changes.
- No "the React Native equivalent of X's Y utility", no "matches Z's animation constants", no
  third-party product names in prose anywhere.
- Docs describe PanelUI's behaviour on its own terms. A reader should never have to know another
  library to understand a page.

Research from them; do not credit them in the artifact. If a reference genuinely needs recording
for future maintainers, it belongs in this file or a commit body — never in shipped code.

## Two ways to consume the library

- **`panelui-native`** — the npm package. The default.
- **`panelui-cli`** — copies a component's source into a project. Backed by the registry at
  `apps/docs/public/r`, generated from `packages/panelui/src` by
  `apps/docs/scripts/build-registry.mjs` and served from panelui.dev.

The registry is generated, never hand-written, so it cannot drift. Two consequences when
changing the library:

- A **new relative import** must be resolvable by the builder, or it throws. Import from
  `../../primitives`, `../../utils/cn`, `../../icons`, `../../native`, `../<component>` or
  `../hooks/<name>` — anything else needs a mapping added to the builder first.
- A **new npm dependency** lands in the registry item automatically, but decide whether it is
  required or optional. Optional means reached through a lazy `require`/`import` inside a
  `try`/`catch`, and it must be listed in `OPTIONAL` in the builder.

## Documentation is part of the change

`apps/docs` is the published documentation site. **A component change is not complete until its
docs page is updated in the same commit.**

- Adding a component → add an entry to `apps/docs/scripts/meta.json` and `usage.json`, then
  regenerate. The MDX file and the group's `meta.json` are written for you.
- Changing a component → update that page's props table, anatomy, variant list and examples. New
  props, renamed variants and changed defaults all count.
- Removing or renaming anything → fix every page that references it.

Props tables are read from the component's actual TypeScript interfaces and their JSDoc in
`packages/panelui/src/components/<name>/index.tsx` — never written from memory. Docs that drift
from the source are worse than no docs, because they are trusted.

**The component MDX is generated, never hand-edited.** `apps/docs/scripts/extract.mjs` reads
the library source into `api.json`; `gen.mjs` merges it with the hand-written `usage.json` and
`meta.json` and writes the MDX. Edit those two JSON files and run
`npm run docs:generate --workspace=docs`, which also rebuilds the registry. See
`apps/docs/scripts/README.md` for what each `usage.json` key becomes.

### meta.json entries: group, addedIn, updatedIn and alpha

A `scripts/meta.json` entry is `[name, summary, keyword]`, optionally followed by an options
object. Four keys live there:

- **`group`** — which sidebar section the page is filed under. Omit it for `components`; pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panel-ui/PanelUI](https://github.com/panel-ui/PanelUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
