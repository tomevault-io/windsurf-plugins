---
trigger: always_on
description: Guidance for coding agents working in this repository. `CLAUDE.md` imports this file, so Claude Code, Codex and anything else that reads `AGENTS.md` all get the same instructions — keep this file self-contained and edit it here rather than adding a second copy.
---

# BoardScribe (by Equalize Digital)

Guidance for coding agents working in this repository. `CLAUDE.md` imports this file, so Claude Code, Codex and anything else that reads `AGENTS.md` all get the same instructions — keep this file self-contained and edit it here rather than adding a second copy.

WordPress plugin — display name "BoardScribe", WP.org slug/text domain `boardscribe`, code prefix `edbs_`/`EDBS_`; git repo `equalizedigital/boardscribe` (renamed from the historical `equalize-digital-meeting-minutes` — old remote URLs redirect), local checkout directory `boardscribe`. Manages meeting minutes as a custom post type (`edbs_boardscribe`), displayed via a `[edbs_boardscribe]` shortcode backed by a public REST endpoint (`/edbs/v1/boardscribe/`), plus a Gutenberg block (`equalize-digital/boardscribe`) that renders through the same shortcode pipeline (moved here from Pro — the block name is stored in existing content and must never change). Native WordPress storage only — no ACF dependency (removed during the restructure).

## This is the free plugin — Pro is a separate sibling repo

This repo is the **free, WordPress.org-distributed** half of a freemium product. The **Pro plugin** lives in a separate repo, checked out locally as a sibling plugin directory (`../boardscribe-pro`, remote `equalizedigital/boardscribe-pro` — renamed from `meeting-minutes-pro` — with its own `CLAUDE.md`); the two repos are developed in lockstep with matching branch names for paired features. This split drives real architecture decisions here:

- Every feature added to this free plugin should ask "how would Pro hook into or override this?" before being considered done.
- Prefer `apply_filters()`/`do_action()` extension points over hardcoding, especially around: CPT registration args, REST route args/query building, row/output formatting, and anywhere shortcode/table markup is assembled.
- If you extract or refactor logic that Pro would need to reuse (e.g. the per-row escaping/formatting logic), expose it as a callable public method rather than leaving it inline — duplicated logic in Pro risks re-diverging from security fixes made here.
- The premium feature list, the free/premium split, and the business context behind that split are tracked privately, not in this public repo. When a change hinges on which side of the free/Pro line a feature falls, ask rather than inferring it from the code.
- Full audit of what's needed for Pro to extend "basically everything" is an ongoing effort — check recent PRs/commit history for the state of this work before assuming a gap exists.

## Directory structure

```
boardscribe.php                        Plugin bootstrap (constants, requires, plugins_loaded hook)
block.json                             Block metadata for equalize-digital/boardscribe (editorScript points at assets/build/block/)
includes/
  Plugin.php                           Singleton; boots all components, fires edbs_loaded (Pro's entry point)
  PostType/BoardScribeCPT.php          CPT registration
  Admin/MetaBox.php                    Native meta box (React app mount point) + registry-driven save handling
  Admin/MetaBoxFieldRegistry.php       Single source of truth for the Meeting Details meta box's fields (see edbs_meeting_meta_fields below)
  Admin/AdminColumns.php                Board Meetings admin list table columns (Meeting Date, Canceled, Agenda, Minutes); Meeting Date is sortable
  Admin/SettingsPage.php               Tabbed settings page (General / Shortcode Builder / Support); the Builder tab enqueues the React builder app
  Helpers/Helpers.php                  UTM link builder for outbound equalizedigital.com links (edition + days_active reporting)
  REST/BoardScribeEndpoint.php      /edbs/v1/boardscribe/ REST route + query/row building
  Shortcode/BoardScribeShortcode.php Shortcode registration, asset enqueuing, instance config
  Shortcode/FieldRegistry.php        Single source of truth for field-backed shortcode attributes (see edbs_shortcode_field_registry below)
  Block/BoardScribeBlock.php           Server-registered block wrapping the shortcode pipeline + editor preview
  Import/CsvImporter.php               CSV bulk-import (Settings page "Import" tab); columns/per-row field-saving are filterable (see edbs_csv_import_columns / edbs_csv_import_row_meta below) so Pro adds its own columns (location, livestream/recording/cc-transcript URLs, documents, date display override, category) without forking the importer
src/js/                                Frontend source modules (ES modules, bundled by webpack)
  index.js                             Entry: exposes window globals, registers table template, bootstraps
  registries.js                        window.edbsExtraColumns/edbsTemplates setup + the template contract docs
  config.js                            Reads window.edbsConfig (i18n, REST base URL)
  templates/table.js                   Built-in "table" display template
  defaults/                            Core fallbacks a template inherits unless it overrides them

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [equalizedigital/boardscribe](https://github.com/equalizedigital/boardscribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
