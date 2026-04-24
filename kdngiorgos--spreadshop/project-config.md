---
trigger: always_on
description: Spreadshop is a **market analysis PoC for Greek resellers**. It ingests supplier product catalogs (XLSX/PDF), scrapes Skroutz.gr marketplace for current selling prices, and produces a profitability report identifying which products to stock.
---

# Spreadshop — Claude Code Project Context

## What This Is

Spreadshop is a **market analysis PoC for Greek resellers**. It ingests supplier product catalogs (XLSX/PDF), scrapes Skroutz.gr marketplace for current selling prices, and produces a profitability report identifying which products to stock.

Target audience: resellers operating in the Greek market (Skroutz.gr is the dominant Greek price-comparison marketplace).

---

## Running the App

```bash
# Install deps (first time only)
pip install -r requirements.txt

# Launch
streamlit run app.py
# → http://localhost:8501
```

**Or use the deploy command:** `/deploy`

### App workflow (5 tabs)

1. **Import** — upload `.xlsx` or `.pdf` supplier files → auto-parsed, summary shown
2. **Products** — browse the full parsed catalog with filters
3. **Scrape** — launch concurrent async tasks using httpx to search Skroutz JSON endpoints for each product; pause/resume/stop controls; results cached
4. **Dashboard** — overview of scraped results
5. **Analysis** — KPI cards + sortable table with margins, competition, opportunity scores; download XLSX report

---

## Proving It Works Without a Browser

```bash
python scripts/demo.py
```

Parses all 3 test files, validates price ratios, runs analysis with mock Skroutz data, exports a sample report to `reports/demo_report.xlsx`. No browser required. Expected output: 207 unique products, 2 parse errors.

---

## Architecture

```
app.py                      Streamlit UI (5 tabs, session state, threading)
parsers/
  base.py                   ProductRecord, SkroutzResult, ParseError dataclasses + parse_file() dispatcher
  xlsx_parser.py            Bio Tonics XLSX (openpyxl, handles category header rows)
  pdf_biotonics.py          Bio Tonics PDF (pdfplumber, comma-decimal prices)
  pdf_viogenesis.py         VioGenesis PDF (pdfplumber + 3-pattern garble decoder)
scraper/
  skroutz.py                httpx async JSON endpoint client, concurrent scraping
  cache.py                  JSON file cache (24h TTL, keyed by barcode)
analysis/
  compare.py                Margin %, opportunity score (0-100), recommendations
  export.py                 Multi-sheet XLSX report with openpyxl styling
scripts/
  demo.py                   Proof-of-concept run: all parsers + mock analysis + export
.streamlit/
  config.toml               Dark theme (#0F1117 bg, #6366F1 indigo accent, monospace font)
cache/                      Auto-created: skroutz_cache.json + upload temp files
reports/                    Auto-created: generated XLSX reports
```

---

## Supplier File Formats

### Bio Tonics — XLSX (`parsers/xlsx_parser.py`)

Sheet name: `Table 1`. Columns:

| Col | Header | Field |
|-----|--------|-------|
| A | ΚΩΔΙΚΟΣ | `code` |
| B | ΠΕΡΙΓΡΑΦΗ | `name` |
| C | ΧΤ | `wholesale_price` (numeric float) |
| D | ΠΛΤ | `retail_price` (numeric float) |
| E | BARCODE | `barcode` (int cast to str) |

Category header rows: text only in col A, C/D/E empty. These set `current_category` for subsequent rows and are not parsed as products.

### Bio Tonics — PDF (`parsers/pdf_biotonics.py`)

Same layout as XLSX but extracted via pdfplumber. Prices are comma-decimal strings (`"7,02"` → `7.02`). One known garbled row (price bleeds into name column) — handled by `_parse_price_str()` which regexes `(\d+)[,.](\d{1,2})` from the cell value.

### VioGenesis — PDF (`parsers/pdf_viogenesis.py`)

Complex garbled format. The PDF was exported with overlapping column layout causing text from adjacent cells to be interleaved. Column mapping (0-indexed):

| Index | Header | Field |
|-------|--------|-------|
| 0 | BARCODE | `barcode` |
| 1 | ΚΩΔ. | `code` |
| 2 | ΠΡΟΪΟΝ | `name` ← **clean, not garbled** |
| 6 | ΚΑΤΗΓΟΡΙΕΣ | `category` (pipe-separated, take first) |
| 9 | Χ.Τ. (24%) | `wholesale_price` ← **garbled** |
| 10 | Λ.Τ. | `retail_price` ← **garbled** |

---

## VioGenesis Price Garble — CRITICAL

The price columns are garbled by interleaved characters from adjacent PDF columns. Three regex patterns decode them. **Do not simplify these without re-testing against the PDF.**

### XT (wholesale) patterns

**Pattern: Main** — covers ~70 products
Raw: `'iasd.gsr2//22w,400p1 €'`  → `22.40`
Encoding: `gsr{d1}//{d2}digits_w,{cents}` → price = `d1 + first_digit(d2) + "." + cents`
Regex: `gsr(\d)//(\d)\d*\w,(\d{2})` → `f"{g1}{g2}.{g3}"`

**Pattern: Single-digit** — covers ~11 products (cheap supplements)
Raw: `'iasd.gsr/9/2,w608p2 €'` → `9.60`
Encoding: `gsr/{d1}/digits,w{cents}` → price = `d1 + "." + cents` (d1 is the full integer)
Regex: `gsr/(\d)/\d+,\w?(\d{2})` → `f"{g1}.{g2}"`

**Pattern: Ads variant** — covers ~2 products (drink powders, high-price items)
Raw: `'ads3/52,0071'` → `35.00`
Encoding: `ads{d1}/{d2}digits,{cents}` → price = `d1 + first_digit(d2) + "." + cents`
Regex: `ads(\d?)/(\d)\d*,(\d{2})` → `f"{g1}{g2}.{g3}"`

### RT (retail) pattern
Raw: `'/eVnito/3Gu7p,e5nl0o e€'` → `37.50`
Strategy: extract all digit characters from the string; the first 4 individual digits form the price as `d1d2.d3d4`.

### Validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kdngiorgos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
