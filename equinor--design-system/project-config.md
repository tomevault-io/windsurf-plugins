---
trigger: always_on
description: Conventions for AI agents working in `apps/design-system-docs` — the public
---

# AGENTS.md — EDS Documentation Site

Conventions for AI agents working in `apps/design-system-docs` — the public
EDS documentation site (eds.equinor.com), built with **Docusaurus 3.10**.
Repo-wide conventions (commits, secrets, formatting, component code style)
live in the root [`AGENTS.md`](../../AGENTS.md); this file covers only what is
specific to this app.

## What this app is

A versioned Docusaurus site documenting EDS. Three doc versions exist:

| Version                             | Content dir                          | URL path                                            | Status                                                   |
| ----------------------------------- | ------------------------------------ | --------------------------------------------------- | -------------------------------------------------------- |
| `current` (labelled **3.0.0-beta**) | `docs/`                              | `/docs/Next/…` (capital N, baked into footer links) | where new work goes                                      |
| `2.0.0-beta`                        | `versioned_docs/version-2.0.0-beta/` | `/docs/2.0.0-beta/…`                                | frozen snapshot, rendered with the redesign              |
| `1.1.0`                             | `versioned_docs/version-1.1.0/`      | `/docs/…`                                           | **frozen archive — never restyle or edit its rendering** |

**Version scoping is the #1 footgun.** Anything that styles doc _content_
must be scoped so 1.1.0 keeps its stock rendering. Current and the frozen
2.0.0-beta both render with the redesign, so every scope names both:

- CSS: pair `html:is([class*='docs-version-current'], [class*='docs-version-2.0.0-beta'])`
  (the redesigned versions) with `html:not([class*='docs-version-'])`
  (unversioned pages: landing, /foundation, /getting-started, /about).
  Per-element rules use `html:where(…)` to keep specificity at 0,0,2 so
  single-class component rules still win. The one exception is the version
  badge in `site-chrome.css`, hidden on current only so frozen pages still
  say which version they are.
- React: the DocItem hero gate checks `REDESIGN_VERSIONS` (`'current'` and
  `'2.0.0-beta'`).
- Freezing another version means adding its `docs-version-*` class and name
  to both lists.
- Chrome (navbar, sidebar, TOC, footer) is deliberately version-independent.

**The current and 1.1.0 paths are pinned explicitly, and both must stay that way.**
`docusaurus.config.ts` sets `lastVersion: 'current'` plus
`'1.1.0': { path: '' }`. Neither is decoration:

- Without `lastVersion`, Docusaurus defaults it to the newest entry in
  `versions.json` (`2.0.0-beta`), which silently makes a frozen snapshot the target
  of every `type: 'docSidebar'` navbar item and of the version dropdown — while
  the footer and landing pages link to `/docs/Next/…`. The site then
  contradicts its own chrome and the redesign is unreachable from the primary
  navigation.
- With `lastVersion: 'current'`, a non-last version takes its version _name_ as
  its path, so `'1.1.0': { path: '' }` is what keeps the archive at `/docs/…`
  instead of relocating it to `/docs/1.1.0/…` and breaking every existing link.

Two sanctioned changes to the archive's rendering, and only these two:

1. It carries Docusaurus's standard "no longer actively maintained" banner,
   because it genuinely is not the latest version. Suppress with
   `banner: 'none'` on the `1.1.0` entry if that is ever unwanted.
2. It has no breadcrumbs. `breadcrumbs: false` is a docs-**plugin** option,
   not a per-version one, so the redesign's choice to drop them necessarily
   applies to the archive too. There is no way to scope it; re-enabling for
   1.1.0 alone would mean a second plugin instance.

Anything else that changes how 1.1.0 renders is a bug.

## Directory map

```
docs/                      current-version content (md/mdx)
versioned_docs/1.1.0/      frozen archive — do not touch
versioned_docs/version-2.0.0-beta/  frozen snapshot — content not edited
src/css/                   the five global stylesheets (see below)
src/components/            shared site components (docs- prefixed CSS)
src/theme/                 Docusaurus swizzles + MDXComponents registry
src/pages/                 unversioned React pages (index, foundation, …)
src/clientModules/         syncColorScheme (data-theme → data-color-scheme),
                           pageTransitions (View Transitions on route change)
scripts/                   check-viewport-overflow.mjs (needs a running
                           server) and check-story-references.mjs (static)
sidebars.ts                hand-maintained; category link docs must NOT be
                           repeated in their own items array
docusaurus.config.ts       aliases + webpack rules (see Config)
```

## Global CSS — five files, strict responsibilities

| File                           | Owns                                                                                                                                                        |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [equinor/design-system](https://github.com/equinor/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
