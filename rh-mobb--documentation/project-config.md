---
trigger: always_on
description: This repository is a **[Hugo](https://gohugo.io/)** static site using the **`rhds`** theme, deployed on **AWS Amplify** with **Pagefind** for client-side search. Agents should treat **[`CONTRIBUTING.md`](./CONTRIBUTING.md)** as the human-facing contribution contract; this file adds automation-oriented defaults and repo-specific gotchas.
---

# Agent guide — MOBB / Red Hat Cloud Experts documentation

This repository is a **[Hugo](https://gohugo.io/)** static site using the **`rhds`** theme, deployed on **AWS Amplify** with **Pagefind** for client-side search. Agents should treat **[`CONTRIBUTING.md`](./CONTRIBUTING.md)** as the human-facing contribution contract; this file adds automation-oriented defaults and repo-specific gotchas.

## Quick facts

| Item | Value |
|------|--------|
| Hugo config | [`config.toml`](./config.toml) |
| Theme | `themes/rhds` (set in config and Make targets) |
| Published subtree | `public/experts` (`publishDir` in `config.toml`; site `baseURL` ends with `/experts/`) |
| Amplify build spec | [`amplify.yml`](./amplify.yml) (Hugo `0.157.0`, Node 24, Pagefind). **`main`** uses `PRODUCTION_BASEURL` (`https://cloud.redhat.com/experts/`); previews/branches use `*.amplifyapp.com`. |
| Amplify redirects | [`customRules.json`](./customRules.json) + Hugo [`_redirects`](./themes/rhds/layouts/_default/index.redir) aliases; merged and synced via [`.github/workflows/sync-amplify-redirects.yml`](./.github/workflows/sync-amplify-redirects.yml). IaC: [`rh-mobb/documentation-infra`](https://github.com/rh-mobb/documentation-infra). |
| Node.js | 24 (see [`.nvmrc`](./.nvmrc); used for Pagefind via `npm ci` / `npx pagefind`) |
| Local preview | `make preview` — see [`Makefile`](./Makefile) |
| Preview + working search | `make preview-search` (builds + Pagefind index; do not use `hugo server -M` / `--renderToMemory` if search must work) |

## Content layout

- **Top-level sections** live under [`content/`](./content/) as directories (for example `rosa/`, `aro/`, `osd/`, `misc/`, `idp/`, `redhat/`, `tags/`).
- **Chapter / section landing pages** use **`_index.md`** (often `archetype: chapter` and `weight` for menu order).
- **Individual guides** use **`index.md`** inside a topic directory (for example `content/rosa/some-topic/index.md`).
- **[`CONTRIBUTING.md`](./CONTRIBUTING.md)** still mentions historical **`/content/docs/...`** paths; prefer the **current** `content/<section>/...` layout unless you are following an explicit maintainer instruction to use `docs/`.
- **[`content/examples/`](./content/examples/)** is a special section whose `_index.md` cascades `draft: true` to all pages within it. Drafts are rendered by `make preview` (which passes `-D`) but excluded from production builds (`hugo --gc --minify`), so example pages are browsable locally but never published. Use this directory for shortcode demos, diagram templates, or formatting examples. Add new pages as `content/examples/<topic>/index.md`; no extra front matter is needed.

### Front matter (guides)

Use YAML front matter at the top of each article. Recommended fields (see CONTRIBUTING for detail):

- `date` — `YYYY-MM-DD`
- `title` — page title
- `tags` — list of strings; **case- and spacing-sensitive**; use established tags from CONTRIBUTING’s taxonomy; coordinate **before** inventing new tags
- `authors` — list of contributors
- `validated_version` (optional) — OpenShift version string when the guide was validated (e.g. `"4.20"`). **Do not** duplicate that disclaimer with a separate `alert` shortcode in the body when this field is set.

Draft content: `draft: true` in front matter; local preview uses `-D` in the Makefile so drafts are visible.

## Markdown and Hugo behavior

- **Goldmark** is configured with `unsafe = true` in [`config.toml`](./config.toml), so raw HTML in Markdown is allowed; still prefer semantic Markdown unless the layout or design system requires HTML/web components.
- **Code fences** are enabled; highlight style and options are under `[markup.highlight]`.
- **Minify**: `disableJS` and `disableJSON` are `true` so builds do not choke on code blocks labeled `json` or `js`. Keep those labels accurate.
- **Shortcodes** ship with the theme under `themes/rhds/layouts/shortcodes/` (for example `alert`, `notice`, `expand`, `tabs` / `tab`, `mermaid`, `include`, `attachments`, `button`, `children`, `swagger`, `math`, `siteparam`). Prefer existing shortcodes over ad-hoc HTML when they fit.
- **`expand` shortcode:** Do **not** rely on it for collapsible sections in this repo. The theme partial uses **jQuery** in its `onclick` handler, but RHDS layouts never load jQuery, so content stays hidden and **clicking the label does not expand**. Fixing that would require a theme change (out of scope for content-only work).
- **Collapsible sections (recommended):** Use native **`<details>`** and **`<summary>`** in content. Browsers handle open/close with no JavaScript. With Goldmark `unsafe = true`, put a **blank line after `</summary>`** so Markdown inside (paragraphs, lists, **tables**) still renders. Example:

  ```html
  <details open>
  <summary>Section title</summary>

  | Column A | Column B |
  |----------|----------|
  | …        | …        |

  </details>
  ```

  Omit the `open` attribute on `<details>` if the block should **start collapsed** instead.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rh-mobb/documentation](https://github.com/rh-mobb/documentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
