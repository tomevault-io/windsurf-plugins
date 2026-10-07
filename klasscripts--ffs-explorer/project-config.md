---
trigger: always_on
description: provides for the explanatory text. Per direct instruction that this
---

# FFS Explorer — codebase map

Desktop forensics tool (PySide6) for browsing iOS/Android Full File System
extractions (Cellebrite / GrayKey zips) without extracting them. Single-window
app; one "case folder" per exhibit holds local caches and results.

**Always run with the project venv:** `venv/bin/python ffs-explorer.py`
(system Python lacks PySide6). Line counts ~23.5k total; `ffs-explorer.py`
alone is ~8.2k — use the section map below instead of reading it whole.

## Architecture in one paragraph

`ffs-explorer.py` builds the main window (`FastZipBrowser`) and owns archive
loading, the file tree/table, filtering, bookmarks, recents, and worker
lifecycle. All heavy or specialised features live in `app/` as **mixins**
(`HexViewerMixin`, `MediaViewerMixin`, `KeywordSearchMixin`,
`ArtifactViewerMixin`, `SqliteViewerMixin`, `SegbViewerMixin`) that
`FastZipBrowser` inherits — so "the hex tab" or "the SEGB tab" means the mixin
file, not the main file. Format *detection and path mapping* (Cellebrite vs GrayKey,
iOS vs Android) live in `app/adapters/`; callers still branch on
`ffs_adapter.format` where behaviour differs (archive classification ~line
842, metadata passes ~1287–1351, integrity sidecar ~2242) — that's expected,
but raw format sniffing belongs only in the adapters. Long work runs in `QThread` workers;
first-open metadata parsing runs in a separate *process* (`ffs_metadata.py`).

## Data flow (opening an archive)

1. `FastZipBrowser.start_loading()` → case dir chosen (`_get_or_ask_case_dir`).
2. `ZipMetadataWorker` (ffs-explorer.py:1462) → `app/ffs_metadata.py
   parse_archive_metadata()` in a child process: central-directory parse,
   `ui_metadata` build, folder tree/sizes; snapshot persisted to case dir
   (msgpack) so re-opens are instant.
3. `FfsAdapter` (`app/adapters/ffs.py`) detects format and maps `ui_path`
   (what the UI shows) ⇄ physical zip path; GrayKey specifics in
   `adapters/graykey.py`; GUID→bundle-id map for `/private/var/mobile/…`.
4. Zip reading: via `zipfile` + `zip_cd_cache.py` (.zcd sidecar so network
   archives open fast). A zip that fails to open (`zipfile.BadZipFile`) is
   treated as corrupted/truncated and surfaces as an error — there is no
   fallback path. Every viewer receives an `app/zip_entry.ZipEntry` and
   never cares whether the entry is stored (direct seek) or deflated
   (zipfile decompress).

## Case folder databases (`app/db_utils.py`)

- `casecache.db` — rebuildable cache: thumbnails, blobs (folder sizes, search
  entry index, `app_intelligence_scan_key` staleness marker — files_indexed
  count + raw_content_enabled state, so toggling raw content on triggers a
  rescan too, not just an archive change),
  header_types, guid_bundle, nested-archive index, `app_intelligence` (see
  `app/app_intelligence.py` below — `evidence_databases_json` since schema
  v7, 2026-08-24, replacing a single `evidence_db_path/bytes/note` triplet;
  see that row's `find_evidence_databases` entry), `app_registry` (schema
  v6, added 2026-08-23 — one row per iOS bundle id: display name, Team ID,
  Bundle/Data container paths, App-Group paths, `has_parser`; see "iOS app
  registry (LaunchServices)" below), `evidence_page_map` (schema v16, added
  2026-09-12 — per-file SQLite page-ownership map, see `sqlite_carve.
  build_page_map`/the "Interpret as SQL Record" Conventions entry below;
  named to avoid SQLite's own reserved `sqlite_`-prefix table-name
  restriction, confirmed directly after `sqlite_page_map` failed with
  "object name reserved for internal use"), `leveldb_search_index` (schema
  v17, added 2026-09-19/20 — one row per real LevelDB/IndexedDB record
  whose decoded content is genuine searchable text, keyed
  `(folder_ui_path, record_index)`; see the "Keyword Search" LevelDB
  Conventions entry below for the full design — staleness tracked via the
  `blobs` table's own generic key/version mechanism
  (`leveldb_search_index_version`), same pattern
  `app_intelligence_scan_key` already uses, rather than this table's own
  schema version). On schema mismatch it is auto-deleted and rebuilt.
- `caseresults.db` — precious results (never auto-deleted): search_index /
  search_results, bookmarks, device_info, run_log (an `artifact_<script_name>`
  row's `parser_version` column records the parser's version — see
  `app/parser_versions.py` in Conventions — at the moment that run happened),
  user SEGB schemas (`segb_schemas`), per-case settings (`case_settings`,
  e.g. AI backend choice), artifact parser output (`artifact_<name>`
  tables), `ai_summaries` (added 2026-08-29 — `script_name` primary key,
  one row per report holding the LAST generated AI Summary: `text`,
  `total_rows`, `chunk_count`, `generated_at`; written by `ai_summary.
  save_summary`, overwritten on each new generate, same as a normal
  `artifact_<name>` table — see the "AI Summary GUI dialog" Conventions
  entry and the app-group-root paragraph right after it), `embedded_media_hits`
  (added 2026-09-22 — one row per confirmed image/video found embedded in
  a SQLite BLOB or plist NSData value, `UNIQUE(source_ui_path, location,
  sha256)` so a re-run of the scan silently skips what it already found
  rather than duplicating; see `app/embedded_media_scan.py` and the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KlasScripts/FFS-Explorer](https://github.com/KlasScripts/FFS-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
