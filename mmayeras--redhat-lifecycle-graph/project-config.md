---
trigger: always_on
description: Instructions for Claude Code when working in this repository.
---

# CLAUDE.md — lifecycle-graph

Instructions for Claude Code when working in this repository.

## What this project is

Standalone Python 3.12 script (`lifecycle-graph.py`) that generates HTML/SVG/PNG Gantt charts for Red Hat product lifecycles. No pip dependencies by default (stdlib only). PyYAML is the only optional dependency, required for config loading.

## Single source of truth

**All product/operator/middleware data lives in `lifecycle-config.yaml`.** The Python script starts with empty dicts and loads everything from YAML at startup. Never add hardcoded dates, fallback dicts, or product configs to the Python file — edit YAML only.

The one exception: `PHASES` dict and `PHASE_KEYS` list in `lifecycle-graph.py` define visual styling for phase types. New phase *types* (not new products) require Python edits there.

## Running the script

```bash
# Generate all charts
python3 lifecycle-graph.py --product all --output-dir docs

# Generate one product (faster for testing)
python3 lifecycle-graph.py --product ocp --output-dir /tmp/test

# Validate API phase_map coverage (RHEL skipped)
python3 lifecycle-graph.py --validate-phases

# Containerized local preview
docker build -t lifecycle-graph:local -f Containerfile .
docker run --rm -p 8080:8080 lifecycle-graph:local

# Install PyYAML first if needed
pip install pyyaml
```

No virtual environment needed. Python 3.12+ required.

## Tests

```bash
python3 -m unittest discover -s tests -v
```

Offline (fetchers monkeypatched). Run after any change to parsers, details data, or rendering; CI runs it before generation. When changing a parser for a new document layout, add a fixture to `tests/test_lifecycle_graph.py` first. Human-oriented contributor guide: `DEVELOPMENT.md`.

## Architecture

```
lifecycle-config.yaml  ──load──▶  PRODUCT_CONFIGS / OPERATOR_CONFIGS / MIDDLEWARE_CONFIGS / _RHEL_MINOR_DATA / _RHEL_MAJOR_DATA
                                         │
                           fetch_lifecycle(cfg)  [Red Hat API, then fallback:]  — skipped for RHEL (use_major_phases)
                                         │
                           build_versions(raw, cfg)  /  build_rhel_major_versions()
                                         │
                           _render_card(versions)  →  HTML Gantt
```

Key functions:
- `_load_external_config()` — loads YAML into runtime dicts at module level
- `_apply_product_overrides()` / `_apply_operator_overrides()` / `_apply_middleware_overrides()` — populate runtime dicts from YAML
- `fetch_lifecycle(cfg)` — calls Red Hat lifecycle API, falls back to `cfg["fallback"]` dict
- `build_versions(raw, cfg)` — filters by min_version, detects EUS, builds phase segments
- `render_combined_html()` — assembles all charts + nav into `lifecycle.html`
- `_generate_lifecycle_about(path)` — renders LIFECYCLE.md → `lifecycle-about.html` (stdlib Markdown converter)
- `_generate_details_page(out_dir, key, cfg, versions)` — z-stream/errata Details page (see below)

## Details pages (z-stream errata)

Products with a `details:` block in YAML (all main products, RHEL included) get two static pages built from one errata fetch: `lifecycle-{key}-details.html` (z-stream releases per minor with errata badges, highlight cards, a From→To version delta filter, release-notes links) and `lifecycle-{key}-timeline.html` (month-grouped vertical timeline of z-stream releases with minor/kind filters). The card gains a `↗ details` link automatically; the two pages cross-link via their topbar.

```yaml
products:
  ocp:
    details:
      errata_query: "OpenShift Container Platform {minor}"   # Hydra search query, {minor} substituted
      release_notes_url: "https://docs.redhat.com/.../{minor}/html/release_notes/"
```

If `errata_query` contains no `{minor}` (e.g. RHOAI, whose synopses read "RHOAI 3.3.5 - …"), a single product-wide query is run and docs are attributed to minors by version parsing; unmatched docs are dropped. Per-minor queries keep unmatched docs as a per-minor "unversioned" list.

Each z-stream body shows **highlight cards** (🔒 Security Fixes / 🔧 Bug Fixes / ✨ Enhancements) built from `* ` bullet lists in `portal_description` — deduplicated across the z-stream's errata. Cards appear only when bullets exist (some products, e.g. RHOAI, have empty descriptions in the search index). Note: docs.redhat.com cannot be scraped at build time (Akamai blocks non-browser clients) — release-notes URLs are link-outs only.

**Feature cards** ("What's new in X.Y", per minor) have two sources, chosen per product in YAML:

- `features_url` (best; OCP): the release-notes *source* asciidoc fetched from the product's public docs repo (openshift/openshift-docs, `enterprise-{minor}` branch — `{minor_dash}` = dots→dashes, `{major}` = major part). The "New features and enhancements" section is parsed into area-grouped title+description entries. The parser (`_parse_adoc_features`) is level-aware (book files and standalone modules), follows one `include::…new-features…` indirection (4.21+ modular books), and handles both `==== Title` headings and `Title::` definition lists. Asciidoc attributes come from `details.attributes_url` plus explicit `details.attributes:` overrides in YAML (e.g. `product-title`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmayeras/redhat-lifecycle-graph](https://github.com/mmayeras/redhat-lifecycle-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
