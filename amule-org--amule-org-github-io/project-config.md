---
trigger: always_on
description: **⚠️ EXCLUDED FOLDERS**: The following folders must be **EXCLUDED** from any analysis, reading, or modification: `node_modules/`, `build/`, `.docusaurus/`.
---

# amule-org.github.io - AGENTS.md

**⚠️ EXCLUDED FOLDERS**: The following folders must be **EXCLUDED** from any analysis, reading, or modification: `node_modules/`, `build/`, `.docusaurus/`.

**⚠️ AGENTS.md** This document is for LLM use. Keep it short — preserve the format and use the minimum number of tokens.

**amule-org.github.io** is the aMule project website, built with Docusaurus v3. Internationalized (i18n).

## Main Libraries

- **Node.js**: `>=24`
- **Framework**: `@docusaurus/core`, `@docusaurus/preset-classic` (`^3.10`)
- **Language**: TypeScript (`tsx` components, `ts` config files)
- **React**: `^18`
- **Syntax highlighting**: `prism-react-renderer`
- **Search**: `@easyops-cn/docusaurus-search-local` — client-side, index built at compile time. Configured in `docusaurus.config.ts` (`themes` array). Add new locales to its `language` array when adding a new i18n locale. Search only works in the production build (`npm run build` + `npm run serve`), not in the dev server (`npm run start`).

## Architecture

**Static site**: `src/pages/index.tsx` (orchestrator) → `src/components/` (section components) → Docusaurus build → GitHub Pages.

**Key files**:
- `docusaurus.config.ts` — site config, navbar, footer, i18n locales, theme, plugins (changelog blog instance)
- `sidebars.ts` — docs sidebar definition
- `src/pages/index.tsx` — homepage, composes section components (Hero/What-is/screenshot inlined here)
- `src/pages/download.tsx` — Download page (`/download`)
- `src/components/<Name>/index.tsx` — one component per homepage section
- `src/components/<Name>/styles.module.css` — scoped styles per component
- `src/css/custom.css` — global CSS variable overrides (color palette)
- `docs/` — English documentation (Markdown)
- `blog/` — Blog posts (`/blog`); `changelog/` — Changelog posts (`/changelog`, second blog plugin instance)
- `i18n/<locale>/` — translations (`code.json` for UI strings; mirrored `docs/`, `blog/`, `changelog/` for content)
- `static/img/` — images (`amule-logo.png`, `social-card.png`, favicons, `screenshots/`, `docs/`)

## Documentation

`docsSidebar` (see `sidebars.ts`) opens with two standalone docs — Overview (`docs/index.md`) and Quick Start (`docs/quickstart-guide.md`) — then **four top-level categories** by audience. Keep them separate — never mix audiences.

- **User Manual** (`docs/manual/`) — install, configure, use, troubleshoot; for basic and expert users. Subdivided into `installation/`, `configuration/` (network config: `directories`, `network-connectivity`, `firewall`, `upnp`, `proxy`, `events`, `macos` + **editable text config files** in `config-files/`: `amule.conf`, `remote.conf`), `interfaces/` (GUI under `gui/`, plus `amuled`/`amuleweb`/`amulecmd`), `utilities/` (standalone helpers), `migration/`, `troubleshooting/`, `faq/`.
- **Developer Guide** (`docs/developer/`) — for aMule developers and advanced integrators: compilation (`compilation/`), debugging, testing, translations, documentation, code style, the **file-format reference** (`file-formats/`: byte layouts of `.met`/`.dat` files), and the **EC protocol**.
- **P2P Networks** (`docs/p2p-networks/`) — general **protocol** description and historical reference: eD2k (`ed2k/`), Kademlia (`kademlia`), `concepts`, `other-networks`. **Do not mix protocol with aMule's concrete implementation** — implementation details belong in the User Manual / Developer Guide and are linked, not embedded.
- **Contributing** (`docs/contributing/`) — `bug-report`.

## Homepage Components

The Hero (logo, tagline, CTA buttons), "What is aMule?" description and the full-width transfers screenshot are inlined in `src/pages/index.tsx`. The remaining sections are components:

| Component | Section |
|---|---|
| `HighlightsSection` | 3.0.0 release highlights grid |
| `FeaturesSection` | Bulleted feature list |
| `ScreenshotsSection` | Screenshot grid with lightbox |

## i18n

- Default locale: `en`. Additional locales: `es`, `fr`, `tr`.
- UI strings (React components): `i18n/<locale>/code.json` — each entry has `message` (translate this) and `description` (context, do not translate).
- **Translated JSON files contain only `message`** — the `description` is translator context and belongs **only** in the English base (`i18n/en/`). Never write `description` into any non-`en` locale file (`code.json`, `navbar.json`, `footer.json`, `current.json`, blog/changelog `options.json`). `write-translations -- --locale <code>` re-adds them and Docusaurus has no option to disable this, so strip them before committing (Weblate keeps the translated files `message`-only via the WebExtension JSON format).
- Docs content: `i18n/<locale>/docusaurus-plugin-content-docs/current/` mirrors `docs/`.
- Blog/changelog content: `i18n/<locale>/docusaurus-plugin-content-blog/` mirrors `blog/`; `i18n/<locale>/docusaurus-plugin-content-blog-changelog/` mirrors `changelog/`.
- Sidebar labels: `i18n/<locale>/docusaurus-plugin-content-docs/current/current.json`.
- Add a new locale: register in `docusaurus.config.ts`, run `npm run write-translations -- --locale <code>`, then translate generated files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amule-org/amule-org.github.io](https://github.com/amule-org/amule-org.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
