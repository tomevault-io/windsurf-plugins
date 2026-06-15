---
trigger: always_on
description: > The canonical reference for AI agents and humans who want to **modify SiteKit itself**. If you want to **build a website with SiteKit**, start at the user-facing [README](README.md) and the [`Plugin/`](Plugin/) subfolder instead.
---

# SiteKit – Contributor reference

> The canonical reference for AI agents and humans who want to **modify SiteKit itself**. If you want to **build a website with SiteKit**, start at the user-facing [README](README.md) and the [`Plugin/`](Plugin/) subfolder instead.

## 1. What SiteKit is

SiteKit is an AI-first Swift static site generator: a phase-oriented build pipeline composed of swappable plugins (Discovery → Loading → Enrichment → Page rendering → System rendering → Output processing, plus content-independent asset teleporting), plus a Claude Code plugin under `Plugin/` that guides agents through building websites for users. The Swift library and the AI plugin live in the same repo so the two evolve together.

## 2. Pipeline

Every build walks a fixed sequence of phases. Each phase is one protocol; each plugin implements one protocol. The phase you want to extend tells you which protocol to conform to and which `SiteBuilder` swap point to use.

| Phase | Protocol | One-line description | `SiteBuilder` swap point |
|---|---|---|---|
| **0. Asset teleporting** | `Teleporter` | Copy assets from source dirs to output | `.teleporter(_:)` |
| **1. Discovery** | `ContentDiscovery` | Find source files for each section | `.contentDiscovery(_:)` |
| **2. Loading** | `Loader<Source, Output>` | Parse Markdown / YAML into typed models | `.articleLoader(_:)`, `.staticPageLoader(_:)` |
| **3. Enrichment** | `Enricher` | Add hreflang, promotions, reading time, etc. | `.enricher(_:)` |
| **4. Per-locale page rendering** | `Page : Renderer` | Render HTML pages with site chrome | `.renderer(_:)` (or `.defaultBlogRenderers()`) |
| **5. System rendering** | `Renderer` (with `RenderScope`) | RSS, sitemap, robots, headers, redirects, etc. | `.renderer(_:)` (same list) |
| **6. Output processing** | `OutputProcessor` | Image variants, font inlining, minification | `.processor(_:)` |

Phase 0 (Teleporter) runs before the content phases and is independent of the content graph – assets never wait on (or feed into) discovery and loading. Phases 1–6 are strictly ordered. Per-locale phases (4 + most of 5) execute once per locale on multilingual sites; `.global` renderers (sitemap index, robots, llms.txt, Cloudflare `_headers`, language redirects) execute exactly once per build regardless of locale count – see `RenderScope`.

### `BuildContext`

The shared, read-only state passed to every Phase 3–6 plugin. Holds the loaded `SiteConfig`, the `ThemeConfig` (if any), the `ContentSection` list (one entry per declared section, each with its loaded `[PageModel]`), `staticPages`, `tags`, `homeContent`, `draftPages`, the locale-aware `URLRouter`, the `UIStrings` bundle for the current locale, and the `outputDirectory` / `projectDirectory` URLs. Plugins read from it; they do not mutate it.

### `SiteBuilder`

A fluent, immutable builder that composes a `BuildPipeline` from the plugins above. Every configuration method (`.renderer`, `.enricher`, `.teleporter`, …) returns a new `SiteBuilder`. Blueprint factory methods like `SiteBuilder.blog(...)`, `.podcast(...)`, `.newsletter(...)`, `.portfolio(...)`, `.docs(...)`, `.docc(...)` pre-compose the default plugin list for a site type. `.run()` reads CLI arguments (`build`, `serve`, `validate`) and executes the pipeline.

### `BuildPipeline`

The executor. Takes the composed plugin list and walks Phases 0–6 in order, dispatching renderers by their declared `scope`. Phase 6 (`OutputProcessor`s) runs after every renderer has written its output files; processors mutate the output directory in place.

## 3. Two-level vocabulary

Two words to keep straight when talking about SiteKit:

- **Blueprint** – a factory-method *recipe* for a site type: `SiteBuilder.blog`, `.podcast`, `.newsletter`, `.portfolio`, `.docs`, `.docc`. Pre-composes the default plugin list and the enricher chain for that kind of site. Blueprints also exist as on-disk site templates under `Plugin/blueprints/` (Blog, IndieDev, Podcast, Newsletter, Portfolio, AppLanding, DocC, Plain, Snippets) – those are the starter sites the Claude Code plugin can clone for a user.
- **Plugin** – any swappable component conforming to a phase protocol (`ContentDiscovery`, `Loader`, `Enricher`, `Page`, `Renderer`, `OutputProcessor`, `Teleporter`). The phase order tells you where it plugs in.

The Claude Code plugin under `Plugin/` is a different sense of "plugin" (the kebab-case Claude Code packaging). Context disambiguates.

## 4. Cross-cutting concerns

Four concerns span multiple phases. For each, here are the phases that contribute and the invariants the framework guarantees.

### SEO / ASO

`PageShell.wrap(content:page:context:)` builds canonical URLs, OG, Twitter Card, JSON-LD, hreflang, and RSS discovery for every page. Per-Page `Renderer`s contribute meta from frontmatter (title, description, image, category). Global `Renderer`s produce `sitemap.xml`, `robots.txt`, and `llms.txt`. The `HreflangEnricher` (Phase 3) populates the per-page `hreflang` table on multilingual sites.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlineDev/SiteKit](https://github.com/FlineDev/SiteKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
