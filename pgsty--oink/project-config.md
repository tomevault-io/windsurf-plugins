---
trigger: always_on
description: Guidance for working in `github.com/pgsty/oink`.
---

# CLAUDE.md

Guidance for working in `github.com/pgsty/oink`.

## Repository boundary

This repository is the OINK Hugo Module: layouts, partials, shortcodes, SCSS,
browser JS, i18n, vendored assets, defaults, migration tools, and theme checks. The
root is the module; there is no site, npm workspace, or generated build input.
Hugo Extended is the build tool.

The sibling `../oink.pgsty.com` repository owns the public bilingual product
documentation, tutorials, examples, case studies, Node/Playwright regression
tests, visual review, and deployment. Its `content/docs/design/` directory is
the only canonical source for bilingual maintainer contracts, decisions,
research, and proposals. Do not recreate `exampleSite/`, `docs/`, `plan/`,
`plans/`, `proposal/`, or another repository-local design tree.

This repository keeps a self-contained synthetic fixture under `tests/site/`
for focused checkers, invalid-input cases, and output goldens. It is not a
public example, integration-test authority, or visual approval surface. The
public site pins a released OINK tag; local cross-repo validation uses an inline
`HUGO_MODULE_REPLACEMENTS` for that command. Never commit a filesystem replace.

`main` is the sole long-lived branch and carries the next release. Immutable
`vX.Y.Z` tags are the stable-release authority and count as published only
after push and Go-proxy verification; do not maintain a second `release`
branch that merely duplicates a tag.

## PRDs and design changes

Create every PRD, RFC, or design proposal in the sibling documentation site as
`content/docs/design/proposals/<slug>.md` plus `<slug>.zh.md`, following the
published lifecycle and template at `/docs/design/proposals/`. A published
proposal remains non-normative until it is implemented and accepted.

When public behavior changes, update the implementation here, its owning
checker, and both language versions of the affected Design contract in the same
delivery. Put stable accepted rationale under the site's `decisions/`, dated
evidence under `research/`, and retired drafts in Git history and
`CHANGELOG.md`—never in a local planning directory.

## Shortest commands

Run the narrow checker that owns a change; `.github/workflows/ci.yml` is the
complete ordered theme-unit suite. Then run integration, browser, accessibility,
responsive, and visual validation against the sibling documentation site.

```sh
python3 bin/check-shell.py               # shell, blog, page-end contracts
python3 bin/check-components.py          # component and render-hook contracts
python3 bin/check-output.py              # fresh strict fixture + output checks
python3 bin/check-namespace.py           # fresh strict fixture + namespace checks
python3 bin/check-goldens.py             # HTML/print/Markdown/RSS/LLMS goldens
python3 bin/check-params.py              # config shapes and warning fallbacks
python3 bin/check-i18n.py                 # locale schema parity
node --test 'tests/js/**/*.test.js'       # browser-runtime units
hugo --source tests/site --printPathWarnings --panicOnWarning
```

Both rendered-output checkers build into a temporary directory when invoked as
shown. Pass `--public tests/site/public` only to reuse an intentionally fresh
fixture that was built separately; this mode does not rebuild that fixture.

Validate and preview the real bilingual site with the current sibling theme
checkout:

```sh
make -C ../oink.pgsty.com check
make -C ../oink.pgsty.com browser
make -C ../oink.pgsty.com dev
```

Migration is dry-run unless `--write` is explicit:

```sh
python3 bin/migrations/oink06.py report --sites <dir>...
python3 bin/migrations/oink06.py migrate --site <dir> [--write]
python3 bin/migrations/oink06.py check --site <dir>
python3 -m unittest discover -s tests/migrations -t .
```

The toolkit rewrites only content Markdown and supported YAML front matter. It
does not edit site configuration, data, layouts, assets, or modules.

Use Hugo Extended 0.160.1 for the compatibility floor. Output checkers accept
`--hugo` and `--public`; reuse a build only when that is the narrowest proof.
`check-goldens.py --update` is an intentional behavior change, never cleanup.

## Non-negotiable contracts

- The current draft contracts are published from the sibling site's
  `content/docs/design/` tree. History belongs in Git and `CHANGELOG.md`, not
  parallel proposal documents.
- Reading shells are `docs`, `book`, `blog`, and `swagger`, plus explicit
  `params.ui.shell_types`. Landing is a layout. Immersive reading is the
  ordinary blog shell with hero/TOC-flow/clouds/sidebar keys, never an
  `article` type or second shell. The blog shell defaults its breadcrumb off;
  `breadcrumb` stays an ordinary key.
- Render content once through `content/render.html` before `scripts.html`.
  Shortcodes and hooks set Page Store capability flags; script selection reads
  them afterwards. Give each new runtime a stable `js/chunks/` target, and put
  language/config identity in that target whenever its generated bytes vary.
- HTML is semantic-first and progressively enhanced. Print is static and
  expanded; Markdown/LLMS contains no component markup; RSS is safe static
  output or explicit omission. Consumers opt into expensive custom outputs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgsty/oink](https://github.com/pgsty/oink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
