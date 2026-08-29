---
trigger: always_on
description: tson.io is the home of the TSON project: a schema and data format for data
---

# TSON.IO — Website

tson.io is the home of the TSON project: a schema and data format for data
interchange, built around strong semantic definitions and closed-world,
verifiable schemas. The README.md contains additional background and details about TSON.

This site hosts the Specification, background research, developer guides, and the
SHA-256-pinned schemas: the **meta-kernel** (the self-referencing meta-schema),
**meta** (the extended meta-schema with the core type constructors), and **core**
(the common types required for data interchange, based on RFC standards).

The specification is a 2026 working draft, with revisions actively underway. When
it is ready, it will be pinned as version 1.

## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.
`astro dev stop` only tracks the most recent instance, so repeated restarts leave orphans
listening on 4321, 4322, 4323 … — check with `lsof -nP -iTCP:4321-4330 -sTCP:LISTEN` if the site
looks stale or broken locally, since the browser is probably pointed at an older one.

Markdown is rendered through a rehype plugin (`src/lib/rehypeTables.mjs`), and Astro caches the
rendered result in `.astro/`. **Editing that plugin changes nothing until the cache is cleared** —
`rm -rf .astro node_modules/.astro` — and a dev server left running across the delete serves
broken pages.

## Dependencies must be declared

Anything imported by `src/` or named in `astro.config.mjs` belongs in `package.json`
`dependencies`, even when it already resolves locally. A local `node_modules` hoists transitive
packages, so an undeclared import works here and fails on Cloudflare, whose clean install from the
lockfile skips optional peers. This has broken deploys twice: `micromark` (imported by
`/research`) and `@astrojs/markdown-remark` (required for `markdown.rehypePlugins` to run at all
under Astro 7's new default processor — its absence silently disables the plugin rather than
erroring).

Note `@astrojs/markdown-remark` is pinned **exactly**, not with a caret: astro declares a
`peerOptional` on one exact version, so `^7.2.0` resolves higher and fails `npm ci` with ERESOLVE.

To verify a dependency change the way Cloudflare sees it, clone to a temp dir, copy in
`package.json`/`package-lock.json`, then `npm ci` and build — `npx astro build` in this working
tree will not catch a missing declaration.

## Revision-scoped `/2026` paths

The spec/schema files live under a revision number so a hash-pinned reference never breaks:
`src/content/2026/{revision}/*.md` (Part 1/2, the guide), `src/content/2026/{revision}/m/*.tn`
(meta-kernel, meta, core), and `src/content/2026/{revision}/fixtures/*.tn` (their resolved-output
fixtures). Every revision stays published at its own path forever; nothing is deleted when the
next one opens.

`src/lib/spec.ts` is the revision registry. **Which** revisions exist is derived from the content
directories, so every page, the sitemap, and the revisions index pick a new one up automatically.
Only two things are declared by hand: `CURRENT_REVISION` (which one is the working draft — the
home page, `/llms.txt`, `/sitemap.xml`, and `public/_redirects` key off it) and `REVISION_NOTES`
(an optional one-line summary per revision for the listing).

Routes:

- `/2026` — a redirect (real 302 via `public/_redirects`, plus a static meta-refresh fallback page
  for local preview) to `/2026/{CURRENT_REVISION}`. Never itself a real, hash-pinnable document.
- `/2026/revisions` — the index of every revision, current and retained. A static route, so it
  wins over `[revision]`; revisions are numeric, so the name can never collide.
- `/2026/{revision}/…` — the revision's own pages. Each renders `RevisionNotice`: a quiet
  "current working draft" line on `CURRENT_REVISION`, and an amber "retained revision" banner
  everywhere else, linking to the same document in the current revision. Retained pages also get
  a `(Revision N)` title suffix so search results distinguish them.

### What a revision directory holds

| Path | Collection | Shown as |
| --- | --- | --- |
| `{revision}/tson-part1-data.md`, `tson-part2-schema.md` | `spec` (`part` set) | "TSON Specification", first |
| `{revision}/tson-guide.md`, any other spec doc | `spec` (no `part`) | "TSON Specification", after the parts |
| `{revision}/*-changelog.md` | `changelog` | "Reports" → "Change Log" |
| `{revision}/reports/*.md` | `reports` | "Reports", split by kind |
| `{revision}/m/*.tn`, `{revision}/fixtures/*.tn` | — (static, see below) | "Schema Source Files", each source with its fixture in parentheses |

**Change logs** are matched by the `-changelog.md` filename suffix, and the `spec` glob excludes
that suffix — so a change log must be named `…-changelog.md` or it will load as a spec document
and fail the schema (it has no `draft`). Frontmatter is `title` plus optional `against`, `status`,
and `inputs` (a list). It renders through the same `[slug].astro` route as the spec documents,
branching on the absence of `draft`.

**Reports** are revision-scoped and are **not** carried forward: a report is written against the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litterat/tson-io](https://github.com/litterat/tson-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
