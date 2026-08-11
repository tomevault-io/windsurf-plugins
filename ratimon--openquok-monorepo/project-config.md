---
trigger: always_on
description: Authoring docs under web/src/content/docs — DocsExternalLink, Badges, Callouts (HTML not Markdown in children), Steps, fenced code (bash for shell commands), agent chat blockquotes, CardGrid/LinkCard
---


# Docs (MD / MDX) authoring

Applies to Markdown and MDsvex files under `web/src/content/docs/`.

## External links (docs pages)

- **In `web/src/content/docs/**`**, use **`DocsExternalLink`** from `$lib/ui/components/docs/mdx/index.js` for **third-party** URLs (Vercel, Supabase, GitHub, npm docs, etc.). It wraps `ExternalLink` with **`not-prose`** and **`text-primary`** + underline so links stay readable inside docs prose (avoids low-contrast inherited `prose-a` styles).
- **Pattern**: `<DocsExternalLink href="https://...">Label</DocsExternalLink>`. Optional props match the base component: `trusted`, `follow`, `ariaLabel`, or `class` to extend styles — see `web/src/lib/ui/components/ExternalLink.svelte` and `web/src/lib/ui/components/docs/mdx/DocsExternalLink.svelte`.
- **`ExternalLink`** remains the low-level primitive (used inside `DocsExternalLink`). Do not use raw `<ExternalLink>` in new docs content unless you have a rare case that must not use docs styling.
- **Do not** rely on raw Markdown **inside** custom MDX component children (e.g. `<Callout>`): see **Markdown inside custom components** below.
- For external URLs in **plain prose** (including **“References”** sections), prefer **`DocsExternalLink`** over raw `<a href="...">` so outbound attributes and contrast stay consistent.

## Links to files in this repository

- Prefer **stable GitHub URLs** to the **default branch** (`main`):  
  `https://github.com/Ratimon/openquok-monorepo/blob/main/<path>`  
  Wrap with `<DocsExternalLink href="...">` when the link leaves the docs site (GitHub is external).
- **Short labels** can use `<code>path/to/file</code>` inside the link, or **`Badge`** with **`variant="path"`** for scan-friendly chips:  
  `<DocsExternalLink href="https://github.com/.../blob/main/backend/vercel.json"><Badge text="backend/vercel.json" variant="path" /></DocsExternalLink>`.
- Keep `docsSite.social.github` in `web/src/data/docs.ts` aligned with the org/repo used in URLs.

## Badges for env vars, paths, and examples (`Badge`)

- Import **`Badge`** from `$lib/ui/components/docs/mdx/index.js` (see `DocsBadge.svelte` for the full variant map).
- Prefer **`<Badge text="BACKEND_DOMAIN_URL" variant="envBackend" />`** (and related variants) over inline **`<code>`** for **environment variable names** in docs: the label is a **string prop**, so underscores and `*` do not trigger MDsveX emphasis bugs, and color coding helps readers distinguish **backend** vs **web (Vite)** vs **runtime** vars.
- **Semantic variants** (use consistently on a page):
  - **`envBackend`** — backend package env (typically no `VITE_` prefix): blue.
  - **`envWeb`** — web / `VITE_*` env: purple. In prose, prefer **`<Badge text="VITE_*" variant="envWeb" />`** (or a specific key) instead of backticks around `VITE_*` / `VITE_` so the label stays visible and MDsveX does not treat `_` as emphasis.
  - **`envRuntime`** — platform flags such as `VERCEL`, `NODE_ENV`, and deployment selectors like `RAILPACK_CONFIG_FILE`: gray.
  - **`path`** — repo-relative paths, folder names, and **HTTP route literals** shown as one chip (e.g. `GET /public/posts/list`, `POST /public/posts`, `DELETE /public/integrations/{id}`): outline/muted. Prefer **`path`** over **`default`** when the label is clearly a URL path or REST line, including in **`web/src/content/docs/cli-usages/`** when tying the CLI to the public API.
  - **`param`** — **CLI flags and long options** (`-c`, `--integrationIds`, `--days`, `-d`), **camelCase JSON / query field names** when shown as chips (e.g. `providerSettingsByIntegrationId`), and **small discrete CLI tokens** in usage lists (e.g. allowed **`--days`** values `7` / `30` / `90`, or top-level verbs like **`upload`** / **`upload-from-url`** on the media CLI page). Use **`param`** so “things you type after the binary” stay visually distinct from command names.
- **Examples**: **`new`** (green) vs **`deprecated`** (red) for **preferred vs discouraged URL shapes** (e.g. with vs without a trailing slash). **`experimental`** (yellow) works well for **npm package names** (e.g. `@sveltejs/adapter-vercel`) when linked to upstream docs.
- **`default`** — generic keys (e.g. `buildCommand`, `installCommand`) when they are not env vars, and **namespaced CLI command names** (`posts:create`, `posts:list`, `analytics:post`, `integrations:list`). Use **`default`** for those; use **`param`** for their **flags** in the same table or sentence.
- **UI labels** (dashboard buttons, provider names, wizard steps): prefer **`Badge`** with **`default`**, **`new`** (positive actions like Save / Enable), **`path`** (navigation crumbs, REST lines, repo paths), **`param`** (CLI flags / field chips in technical docs), or **`experimental`** (third-party package names per above)—keeps emphasis visible without fragile `**bold**` in MDX.

## Underscores and `*` in environment variable names (MDsveX)

- Inline Markdown **backticks** around names like `` `NODE_ENV` `` or `` `BACKEND_DOMAIN_URL` `` can be parsed as **emphasis** (`_…_` / `*…*`), which breaks the Svelte compile (`element_invalid_closing_tag`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
