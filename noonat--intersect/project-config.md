---
trigger: always_on
description: This repository is a tutorial that happens to also be a library. `noonat/intersect`
---

# Working on intersect

This repository is a tutorial that happens to also be a library. `noonat/intersect`
is a set of 2D collision detection routines — AABB and circle tests, intersection
and swept — written as a literate document and published to
<http://noonat.github.io/intersect>.

The tutorial is the point. The library is what falls out of it. Whenever the two
pull in different directions, readability of the tutorial wins.

## Working agreements

These come first because they are the easiest to get wrong.

**Never commit without approval, and never push without a second approval.**
They are two separate gates. Draft the commit message, show it, and wait. Once
the message is approved, ask again before pushing. "Yes, commit that" is not
permission to push.

**No bylines on commits.** No `Co-Authored-By`, no `Generated with`, no tool
attribution of any kind in the message. Subject lines are short and written in
the imperative. Many are subject-only — a body is for when there is genuinely
something to explain, not a habit. Semicolons in subject lines have been asked
against specifically.

Recent history is a fair guide to the house style:

```
Minor corrections to captions and prose
Drop code blocks below floated figures
Scale each example to its own view, and animate on demand
Draw the arrowheads as filled triangles
Colour the code, and keep the examples on a dark plate
Redesign the tutorial page
```

## The one rule about editing

**`src/intersect.ts.md` is the source of truth for both the prose and the
library code.** Every code block in that Markdown file is extracted, in order,
to produce `src/intersect.ts`.

Never edit `src/intersect.ts` directly. It is generated, it is committed, and
your edit will be silently reverted by the next build. The same goes for
`index.html`, `lib/`, `docs/bundle.*`, and `docs/docco.css`.

To change the library, edit the code block inside the Markdown. To change the
page's design, edit `template/docco.css` (which is copied to `docs/docco.css`)
or `template/docco.jst`.

## Build pipeline

`npm run build` runs `build.js`, which does four things:

1. **`compileSource()`** — parses `src/intersect.ts.md` with markdown-it, keeps
   only the code blocks, concatenates them, runs the result through Prettier,
   and writes `src/intersect.ts`. A blank line is inserted after any block that
   ended in a closing brace, because Markdown has no way to express the blank
   lines you would want between them.
2. **`compileHTML()`** — runs `docco` over the Markdown with
   `template/docco.jst` and `template/docco.css`, inlines the SVG figures (see
   below), formats with Prettier, and writes `index.html` at the repository
   root. docco's own output file is deleted afterwards.
3. **`compileLibrary()`** — `tsc` over `src/intersect.ts` into `lib/`.
4. **`compileExamples()`** — type checks `src/examples.ts` against
   `tsconfig.examples.json` (esbuild only strips types, so without this the
   examples are checked by nothing), then bundles to `docs/bundle.js`. The
   KaTeX stylesheet is imported by `examples.ts`, so esbuild also emits
   `docs/bundle.css` and the 60 hash-named KaTeX font files in `docs/`. Those
   hashed files are real build output, not cruft — leave them alone.

### Build output is committed, and CI checks it

The `build` job in `.github/workflows/ci.yml` runs `npm ci` (whose `prepare`
script builds everything) and then fails if `git status --porcelain` is
non-empty. **If you touch anything that feeds the build, run `npm run build`
and commit the result in the same commit.** A change to the Markdown, the
template, the CSS, the examples, or the SVGs all qualify.

The `visual` job in the same file runs the Playwright suite against the
committed page. It installs with `--ignore-scripts` precisely so that it does
not rebuild: the `build` job above is what checks the committed output matches
its source, and this one is then testing the page the repository actually
publishes.

### Commands

```
npm run build     # regenerate everything
npm test          # jest (66 tests), eslint, and the visual tests' type check
npm run test:visual   # Playwright — needs a browser, see "Visual tests"
npm run clean     # remove generated files
```

`npm test` is a few seconds, needs neither a browser nor the network, and is
the one to run by habit. The visual tests are deliberately not part of it.

## The page design

The rendered page has a single design token system in `template/docco.css`,
declared three times so that theming works in all three states:

- bare `:root` — the light palette, the full set
- `@media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) }` —
  system dark, unless the reader has explicitly chosen light
- `:root[data-theme="dark"]` — the toggle, which must beat the media query

A token defined only inside a media query has no light value and will break.
Define everything on bare `:root`, then override in the two dark blocks.

The token groups are:

- **Page** — `--paper --ink --hard --gray --soft --panel`
- **Diagrams** — `--d-ground --d-world --d-edge --d-quiet --d-query --d-clear
  --d-correct --d-collide --d-frame --d-label`
- **Syntax** — `--code-comment --code-keyword --code-string --code-number
  --code-title --code-type`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noonat/intersect](https://github.com/noonat/intersect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
