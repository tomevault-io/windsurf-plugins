---
trigger: always_on
description: Hwaro is a fast, lifecycle-driven static site generator written in Crystal. Features a hook-based build pipeline, pluggable processors, multi-layer caching, and parallel content processing.
---

# Hwaro - Agent Instructions

## Overview
Hwaro is a fast, lifecycle-driven static site generator written in Crystal. Features a hook-based build pipeline, pluggable processors, multi-layer caching, and parallel content processing.

## Build & Run
```bash
just build          # shards install && shards build → bin/hwaro
just test           # crystal spec (unit + functional + content)
just fix            # crystal tool format
just dev            # Serve docs site locally (bin/hwaro serve -i docs)
just clean          # Remove bin/, lib/, stb_impl.o
```
Dependencies (shard.yml): `markd` (Markdown), `toml` (TOML parsing), `crinja` (Jinja2 templates), `emoji`.

## Directory Structure
```
src/
  cli/              # Runner, metadata (FlagInfo), commands/, commands/tool/
  config/options/   # Typed option classes per command (BuildOptions, ServeOptions, etc.)
  content/
    processors/     # Base, Markdown, Template, HTML, XML, JSON, Image, SyntaxHighlighter
    filters/        # Crinja filters: string, collection, date, html, url, i18n, math, misc
    hooks/          # Markdown, SEO, Taxonomy, Asset, PWA, AMP, OgImage, Image hooks
    seo/            # Feeds, Sitemap, Robots, JsonLD, Tags, LLMs, PWA, OgPngRenderer
    pagination/     # Paginator, Renderer
  core/
    lifecycle/      # Manager, Hooks (HookResult/Hookable/HookDSL), Phases, BuildContext
    build/          # Builder, CacheManager, ShortcodeProcessor, Parallel, phases/
  models/           # Page, Section, Site, Config, Toc, Deployment
  services/
    scaffolds/      # Registry + 8 built-in (simple/bare/blog/docs/book + dark variants) + remote
    importers/      # WordPress, Jekyll, Hugo, Notion, Obsidian, Hexo, Astro, Eleventy
    exporters/      # Jekyll, Hugo
    server/         # Dev server with live reload and file watching
    defaults/       # Default content, templates, config
  utils/            # Logger, PathUtils, TextUtils, CrinjaUtils, Profiler, minifiers
  ext/              # Crinja patches, stb_image bindings, fonts
spec/
  unit/             # Component-level tests (~150 files)
  functional/       # End-to-end build tests (~20 files, use build_helper.cr)
  content/seo/      # SEO output validation
```

## Architecture

### Lifecycle Pipeline
8 sequential phases, each with `before`/`after` hook points (16 total):
```
Initialize → ReadContent → ParseContent → Transform → Render → Generate → Write → Finalize
```
- **BuildContext**: Shared state container (pages, sections, site, config, cache) passed through all phases.
- **Hook registration**: Modules implement `Hookable` interface, register via `Manager#on(HookPoint, priority:, name:)`.
- **HookResult**: `Continue` | `Skip` | `Abort`.

### Registry Pattern
Used for: Processors (`ContentProcessors::Base`), Commands (`CommandRegistry`), Scaffolds (`Scaffolds::Registry`), Hooks (`Manager`).

### CLI
- Commands define `NAME`, `DESCRIPTION`, `FLAGS` (array of `FlagInfo`), `POSITIONAL_ARGS`.
- Tool subcommands: list, convert, check-links, stats, validate, unused-assets, platform, doctor, import, export, agents-md, ci.
- Shell completion auto-generated from `CommandRegistry` metadata.
- `doctor` is a top-level alias for `tool doctor`.

### Caching (Multi-layer)
1. **Build cache** (`.hwaro_cache.json`) - File mtime + content hash.
2. **Template compilation cache** - Compiled Crinja AST keyed by `UInt64`.
3. **Crinja value caches** - Per-page, per-section, per-series, per-ancestor. Cleared at phase transitions.
4. **Site lookup indices** - `pages_by_section`, `sections_by_parent`, `sections_by_name`.

### Models
- **Page**: 50+ properties (title, date, draft, tags, content, url, word_count, reading_time, series, authors, extra, etc.).
- **Section**: Extends Page with paginate, sort_by, reverse, transparent, subsections.
- **Site**: Aggregator with lookup indices, taxonomy maps, data/authors hashes.
- **Config**: Nested structures for feeds, sitemap, robots, search, SEO, image processing, multilingual.

## Development Guide

### Adding a New Processor
1. Inherit `Hwaro::Content::Processors::Base`, implement `process(context : ProcessorContext) : ProcessorResult`.
2. Register in processor `Registry`.

### Adding a New Command
1. Create class in `src/cli/commands/` with `NAME`, `DESCRIPTION`, `FLAGS` (using `FlagInfo`), and `run(args)`.
2. Register in `src/cli/runner.cr` via `CommandRegistry.register(metadata, &handler)`.

### Adding a New Hook
1. Implement `Hwaro::Core::Lifecycle::Hookable` with `register_hooks(manager)`.
2. Register in `src/content/hooks.cr`.

### Adding a Config Option
Update **all** of the following:
1. `src/models/config.cr` — property, default, and loader.
2. `src/services/config_snippets.cr` — shared TOML snippets (accepts `commented` param for scaffold vs doctor variants).
3. `src/services/scaffolds/base.cr` — if scaffold-only section.
4. `spec/unit/config_spec.cr` and relevant feature specs.

## Coding Patterns

### Security
- **HTML/XML output**: `Utils::TextUtils.escape_xml(value)` or `HTML.escape(value)`.
- **Inline JS**: Escape `</` → `<\/` in JSON data to prevent `</script>` breakout.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hahwul/hwaro](https://github.com/hahwul/hwaro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
