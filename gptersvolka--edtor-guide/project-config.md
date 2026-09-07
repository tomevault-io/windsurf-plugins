---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gpters Content Intelligence** is a data-driven content performance analysis system that learns successful writing patterns from blog post analytics. It uses Google Sheets as the single source of truth, processes Mixpanel/GSC data, extracts features from content, and generates unified writing guidelines through ML modeling.

Key constraint: All performance metrics are calculated using a **fixed 30-day window from the publication date** (`published_date + 30 days`). Posts with less than 30 days since publication are excluded from evaluation.

## Core Architecture

### Data Flow Pipeline

```
1. posts_input (user input)
   ↓
2. crawler.py → content_parsed (HTML parsing, feature extraction)
   ↓
3. mixpanel_raw / gsc_raw (manual CSV upload)
   ↓
4. mixpanel_processor.py / gsc_processor.py → metrics_30d (30-day window aggregation)
   ↓
5. score_calculator.py → scores (onsite_score, search_score)
   ↓
6. extractor.py → features (title_length, h2_count, etc.)
   ↓
7. baseline_model.py → model_insights (ML feature importance)
   ↓
8. pattern_extractor.py + guideline_merger.py → guidelines_master (unified rules)
```

### Google Sheets as Data Store

All intermediate and final outputs live in Google Sheets. The `SheetsConnector` class (`src/utils/sheets_connector.py`) wraps `gspread` and provides:
- `read_sheet_to_dataframe(sheet_name)`: Load sheet as pandas DataFrame
- `write_dataframe_to_sheet(df, sheet_name)`: Write DataFrame to sheet (clears first by default)
- `append_rows_to_sheet(rows, sheet_name)`: Append-only for raw data sheets

**Critical rule**: Never modify `mixpanel_raw` or `gsc_raw` sheets directly. These are append-only.

### Module Structure

- **`data_collection/`**: Web scraping (BeautifulSoup), CSV processing (Mixpanel/GSC), 30-day window filtering
- **`feature_extraction/`**: Parse HTML content and extract structured features (see SPEC.md section 7)
- **`modeling/`**: Score calculation, ML models (start with LinearRegression/DecisionTree for small samples)
- **`guideline_generation/`**: Top corpus selection, LLM-based pattern extraction, rule merging with confidence scoring
- **`utils/sheets_connector.py`**: All Google Sheets I/O goes through this module

## Common Commands

### Environment Setup
```bash
# Create virtual environment (Python 3.9+)
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # macOS/Linux

# Install dependencies
pip install -r requirements.txt
```

### Initialize Google Sheets Structure
```bash
# First time only: creates 9 required sheets with headers
python scripts/init_sheets.py
```

### Data Processing Pipeline (sequential execution)
```bash
# 1. Crawl posts from posts_input sheet
python src/data_collection/crawler.py

# 2. Process Mixpanel CSV (after uploading to mixpanel_raw sheet)
python src/data_collection/mixpanel_processor.py

# 3. Process GSC CSV (after uploading to gsc_raw sheet)
python src/data_collection/gsc_processor.py

# 4. Calculate scores
python src/modeling/score_calculator.py

# 5. Extract features
python src/feature_extraction/extractor.py

# 6. Run ML model (Phase 2+)
python src/modeling/baseline_model.py

# 7. Generate guidelines
python src/guideline_generation/auto_updater.py
```

### Configuration

Before running any scripts, create `config/settings.py` from template:
```bash
cp config/settings.py.template config/settings.py
# Edit: set CREDENTIALS_FILE, DRIVE_FOLDER_ID, SPREADSHEET_NAME
```

Google Sheets API setup requires:
1. Enable Google Sheets API and Google Drive API in Google Cloud Console
2. Create Service Account and download JSON key to `config/credentials.json`
3. Share Google Drive folder with Service Account email
4. Set `DRIVE_FOLDER_ID` in `config/settings.py`

## Critical Implementation Rules

### 1. 30-Day Window Enforcement
Every metric calculation must filter by:
```python
window_start = published_date
window_end = published_date + timedelta(days=30)
# Filter: window_start <= event_timestamp < window_end
```

### 2. Sheet Schema Rules
When adding new sheets:
- Add description in rows 1-2
- Use snake_case for sheet names
- Always include `post_id` as the join key
- Update `SHEET_SCHEMAS` in `scripts/init_sheets.py`

### 3. Data Source Constraints
- **Mixpanel**: NO API access. Only CSV export → `mixpanel_raw` upload
- **GSC**: Only URL-level data. No query-level data
- **posts_input**: User manually enters URLs and `published_date`. Never auto-crawl the site

### 4. Guideline Generation Strategy
- Analyze **onsite** (Mixpanel) and **search** (GSC) performance separately
- Extract patterns from top performers in each category
- Merge into `guidelines_master` with:
  - `scope`: onsite/search/merged
  - `confidence`: 0-1 (based on sample size)
  - `evidence_post_ids`: list of supporting posts
- Confidence formula: `sample_size / (sample_size + 10)` for small samples

### 5. Feature Extraction
All features defined in SPEC.md section 7 must be extracted from HTML:
- Structural: `h2_count`, `h3_count`, `section_count`, `bullet_count`
- Content: `title_length`, `intro_length`, `word_count`, `reading_time_est`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gptersvolka/edtor_guide](https://github.com/gptersvolka/edtor_guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
