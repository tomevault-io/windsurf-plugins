---
trigger: always_on
description: The DOJ released 2.73 million pages of Epstein case files on January 30, 2026, across 12 datasets at [justice.gov/epstein](https://www.justice.gov/epstein/). This repository contains **searchable databases** built from those files — 1,385,916 documents and 2,771,231 pages of extracted text, fully indexed for full-text search.
---

# Epstein Files — Data Repository

## What This Is

The DOJ released 2.73 million pages of Epstein case files on January 30, 2026, across 12 datasets at [justice.gov/epstein](https://www.justice.gov/epstein/). This repository contains **searchable databases** built from those files — 1,385,916 documents and 2,771,231 pages of extracted text, fully indexed for full-text search.

The databases are hosted as GitHub releases because they're too large for the repository itself (~17 GB uncompressed total).

## First-Time Setup

### 1. Check for sqlite3

```bash
sqlite3 --version
```

If missing: Mac/Linux usually have it pre-installed. On Ubuntu/Debian: `sudo apt install sqlite3`. On Windows: download from [sqlite.org/download.html](https://www.sqlite.org/download.html) and add to PATH, or use `winget install SQLite.SQLite`.

### 2. Download the databases

Use `gh release download` if available, otherwise `curl -LO`. There are two releases to download from.

#### Full text corpus (v5.0) — the main database

```bash
# Download both parts (~2.3 GB total compressed)
gh release download v5.0 --repo rhowardstone/Epstein-research-data --pattern "full_text_corpus.db.gz.*"

# Reassemble and decompress
cat full_text_corpus.db.gz.part_aa full_text_corpus.db.gz.part_ab > full_text_corpus.db.gz
gunzip full_text_corpus.db.gz
# Result: full_text_corpus.db (~6.3 GB)

# Clean up parts
rm full_text_corpus.db.gz.part_aa full_text_corpus.db.gz.part_ab
```

#### All other databases (v5.1)

```bash
gh release download v5.1 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gunzip concordance_complete.db.gz alteration_results.db.gz image_analysis.db.gz
```

#### Remaining databases (v4.0)

```bash
# Smaller databases not yet consolidated into v5.1
gh release download v4.0 --repo rhowardstone/Epstein-research-data --pattern "*.db.gz"
gh release download v4.0 --repo rhowardstone/Epstein-research-data --pattern "*.db"
gunzip *.db.gz
```

If `gh` is not available, download manually from:
- https://github.com/rhowardstone/Epstein-research-data/releases/tag/v5.0
- https://github.com/rhowardstone/Epstein-research-data/releases/tag/v5.1
- https://github.com/rhowardstone/Epstein-research-data/releases/tag/v4.0

### 3. Verify setup

```bash
sqlite3 full_text_corpus.db "SELECT COUNT(*) || ' documents, ' || (SELECT COUNT(*) FROM pages) || ' pages' FROM documents;"
```

Expected output: `1385916 documents, 2771231 pages`

---

## Database Reference

### `full_text_corpus.db` (6.3 GB) — Primary search database

The main database. Every page of every document, with full-text search.

**Tables:**

```sql
-- documents: one row per PDF/document
-- Columns: id, efta_number (unique), dataset (1-12, 98, 99), file_path, total_pages, file_size
SELECT * FROM documents WHERE efta_number = 'EFTA00074206';

-- pages: one row per page of each document
-- Columns: id, efta_number, page_number, text_content, char_count
SELECT * FROM pages WHERE efta_number = 'EFTA00074206' ORDER BY page_number;

-- pages_fts: FTS5 full-text search index on pages
-- Searchable columns: efta_number, text_content
SELECT * FROM pages_fts WHERE pages_fts MATCH 'search terms';
```

### `concordance_complete.db` (729 MB) — Cross-reference metadata

DOJ production metadata: original filenames, email headers, folder paths, dates, custodians, MD5 hashes.

**Key columns in `documents` table:** bates_begin, bates_end, original_filename, document_extension, original_folder_path, author, custodian, date_sent, email_from, email_to, email_cc, email_subject, efta_number

Also has: `email_threads`, `folder_inventory`, `extraction_stats`, `cross_references`

### `redaction_analysis_v2.db` (1.0 GB) — Redaction patterns

2.6 million detected redaction rectangles across 850K documents. Includes OCR of text under improperly applied redactions.

**Tables:** `redactions` (efta_number, page_number, redaction_type, ocr_text, confidence), `document_summary`, `reconstructed_pages`

### `alteration_results.db` (557 MB) — DOJ document alteration tracking

212,730 change units tracking differences between original and current versions of DOJ-hosted documents.

**Table:** `altered_files` (efta_number, dataset, diff_type, categories, removed_names_json, llm_classification, llm_sensitivity, llm_justification, anomaly_flag)

### `image_analysis.db` (762 MB) — Extracted images

92,095 images extracted from PDFs across all 14 datasets (DS1-12, DS98, DS99), analyzed with Qwen2-VL-7B vision model. FTS5 searchable by description, people, objects, and setting.

**Table:** `images` (image_name, efta_number, page_number, analysis_text, people, text_content, objects, setting, activity, notable, analyzed_at)

### `handwriting_transcriptions.db` (248 KB) — Handwritten document transcriptions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhowardstone/Epstein-research-data](https://github.com/rhowardstone/Epstein-research-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
