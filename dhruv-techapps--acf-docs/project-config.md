---
trigger: always_on
description: The user documentation for Auto Clicker AutoFill — how to install it, how to use it, and what every feature does. Astro site published at **getautoclicker.com/docs/5.x**.
---

# acf-docs

The user documentation for Auto Clicker AutoFill — how to install it, how to use it, and what every feature does. Astro site published at **getautoclicker.com/docs/5.x**.

**Updating these docs is part of every release.** Any user-facing change ships with its doc update, the same way it ships with a release-note post in `acf-blog`.

Only the current version (`5.x`) is maintained. Older versions are not updated — they survive as `aliases` redirects on the pages that replaced them.

## Layout

```
config.yml                     — site-wide config (versions, social, Algolia app id, GTM)
site/astro.config.ts
site/content.config.ts         — the Zod front-matter schema (authoritative)
site/data/sidebar.yml          — navigation
site/src/content/docs/<section>/<page>.mdx
site/src/components/shortcodes/ — MDX components usable in pages
```

Sections: `getting-started`, `side-panel`, `automations`, `automation`, `step`, `step-value`, `settings`, `extension`, `userscript`, `about`, `faq`.

Note `automations` (plural — managing the list: export, import, explore, reorder) and `automation` (singular — configuring one: url, settings, schedule, loop, monitor) are different sections. Easy to put a page in the wrong one.

## Front matter

`site/content.config.ts` is the schema and it is enforced at build time. Valid keys:

| Key                                                     | Notes                                                                                                                                     |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `title`                                                 | **required**                                                                                                                              |
| `description`                                           | **required**                                                                                                                              |
| `subscription`                                          | `PLUS` or `PRO` — renders the tier badge. Omit for free features.                                                                         |
| `tags`                                                  | keyword array — emitted as `docsearch:tags` + `keywords` meta and indexed by Algolia. All 68 pages carry them; keep new pages consistent. |
| `added`                                                 | `{ version: '5.0.0', show_badge?: bool }` — "new in" badge                                                                                |
| `aliases`                                               | string or array — redirects from old URLs; carry these forward, they're how 3.x/4.x links keep working                                    |
| `toc`                                                   | bool                                                                                                                                      |
| `thumbnail`, `direction: 'rtl'`, `extra_js`, `sections` | occasional use                                                                                                                            |

Anything else is **silently dropped** — Zod strips unknown keys rather than erroring. See "Known gaps" about `tags`.

## Adding a page

1. Create `site/src/content/docs/<section>/<page>.mdx` with valid front matter.
2. **Add it to `site/data/sidebar.yml`** under the right group, by `title`.

`DocsSidebar.astro` throws if `sidebar.yml` references a page that doesn't exist — a typo in either place fails the build, so the two must agree.

## Writing pages

- `[[config:docs_version]]` interpolates from `config.yml` — use it in asset paths rather than hardcoding `5.x`: `<img src="/docs/[[config:docs_version]]/assets/img/loop.png" />`
- Shortcodes live in `site/src/components/shortcodes/`: `Callout`, `Code`, `Table`, `BsTable`, `Video`, `AddedIn`, `DeprecatedIn`, `SubscriptionBadge`, `Placeholder`, `IncludeMdx`, `Example`, `ExampleAutomation`.
- `<ExampleAutomation file="loop" plus>` offers a downloadable sample automation and points at **test.getautoclicker.com** (the `acf-test` page) as the place to try it. The `plus` prop labels it as needing the PLUS plan.

## Never link to PRs or branches

**Don't reference pull requests, branches, or commits in a doc page.** Some work happens in private branches, so those links are dead or inaccessible for readers. Describe behaviour in user-facing terms. The same rule applies in `acf-blog`.

## Local development

```bash
npm run docs-serve     # astro dev on port 9001
npm run docs           # build + lint (prettier check + HTML validation)
```

In dev the site origin is `http://localhost:9001`; in production it's `baseURL` from `config.yml`.

## Deploy

Push a tag `v*` → `gh-pages.yml` builds, validates HTML (`vnu`, needs Java), runs `linkinator`, and publishes to GitHub Pages. Nothing deploys on merge to main.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dhruv-Techapps/acf-docs](https://github.com/Dhruv-Techapps/acf-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
