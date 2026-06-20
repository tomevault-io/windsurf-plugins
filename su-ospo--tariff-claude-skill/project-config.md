---
trigger: always_on
description: Digitize tariff document PDFs into structured Excel spreadsheets and CSVs. Use this skill whenever the user uploads a tariff PDF, tariff schedule, customs duty document, harmonized tariff schedule (HTS/HTSUS), trade regulation PDF, Schedule A (Classification of Imports), or any PDF containing tariff rate tables, duty rates, or customs classification data. Also trigger when the user mentions 'tariff digitization', 'tariff table extraction', 'convert tariff PDF', 'tariff spreadsheet', 'Schedule A'
---


# Tariff Document Digitizer

Convert tariff document PDFs into clean, structured Excel (.xlsx) and CSV files by extracting and reconstructing the tables they contain. Designed for academic research on historical tariff schedules, particularly the U.S. Classification of Imports (Schedule A), but adaptable to any tariff document format.

## Core Principle: Complete Replica

**Extract ALL columns** from the source document, not just a subset. The final spreadsheet should be a complete replica of the input PDF's tabular data. Every column visible in the PDF table should appear in the output.

---

## Training Data

Before starting any extraction, check for bundled training data to understand the baseline output format:

- **`references/training_data.csv`** — A clean, pre-parsed CSV containing correctly extracted tariff data from research. This is your ground truth for description formatting and hierarchy conventions.
- **`references/training_data.pdf`** — The original source PDF that the CSV was extracted from. Use this to understand the kinds of layouts you'll encounter.

At the start of Phase 2, load the CSV training data if available:
```python
import pandas as pd
import os
skill_path = "<skill_path>"
training_csv = os.path.join(skill_path, "references", "training_data.csv")
if os.path.exists(training_csv):
    training_df = pd.read_csv(training_csv)
    print(training_df.head(10))
    print(training_df.columns.tolist())
```

Use the training data to:
1. **Learn the baseline column structure and description conventions.** The training CSV may contain a simplified subset of columns (e.g., `Schedule A Commodity Number`, `Commodity Description`, `Tariff Paragraph`). Source documents typically contain additional columns. **Always extract ALL visible columns from the source document** — the training data represents a baseline, not the complete target schema.
2. **Understand formatting conventions.** Commodity numbers use the format `NNNN NNN` (e.g., "0010 600"). Descriptions use colon-separated hierarchies. Tariff paragraphs can be simple numbers ("701"), compound references ("701, §2491 (c) I.R.C"), or subsection references ("708 (a)").
3. **Validate your extraction.** After extracting data, programmatically compare any overlapping commodity numbers between your output and the training data.

---

## Document Edition Detection

Historical U.S. tariff schedules come in different editions with distinct formats. **Identify the edition first** — it determines column structure, number format, and output schema.

### 1950 Edition
- Commodity numbers in `XXXX XXX` format with a space (e.g., `0010 600`)
- **7 columns** with two separate Rate of Duty sub-columns
- Date printed: "August 1, 1950"

### 1939 Edition
- Commodity numbers use period notation (e.g., `0010.6`, `*0046.45`)
- **6 columns** with a single Rate of Duty column
- Contains Revenue Act references

### Other / Unknown Editions
- Adapt the column schema to whatever the source document contains
- Follow the same hierarchy and extraction principles below

---

## Output Format by Edition

### 1950 Edition Columns (7 columns)

| Column | Width | Description |
|--------|-------|-------------|
| **Schedule A Commodity Number** | 25 | `XXXX XXX` format (e.g. `0010 600`) |
| **Commodity Description** | 120 | Fully-qualified hierarchical description with colon separators |
| **Economic Class** | 15 | The parenthetical class number, e.g. `(2)`, `(4)`, `(9)` |
| **Unit of Quantity** | 20 | e.g. `Lb`, `No`, `Gal`, `Piece; Lb`, `Doz`. Keep the cattle `v` superscript inline (see **Unit of Quantity** below) |
| **1930 Tariff Act (except as noted)** | 40 | Statutory rate, including `(Sec. 336)` notes |
| **Trade Agreement** | 70 | GATT and country-specific concession rates with abbreviations |
| **Tariff Paragraph** | 25 | Including compound references like `701, 2491 (c) I.R.C.` |

### 1939 Edition Columns (6 columns)

| Column | Width | Description |
|--------|-------|-------------|
| **Economic Class** | 15 | Numeric class (e.g. `2`, `4`, `5`) |
| **Schedule A Commodity Number** | 25 | Converted from period to `XXXX XXX` format |
| **Commodity Description** | 120 | Fully-qualified hierarchical description |
| **Unit of Quantity** | 25 | e.g. `Lb.......1`, `No......20`, `Gal.......7`. Keep the cattle `v` superscript inline (see **Unit of Quantity** below) |
| **Rate of Duty** | 60 | Full duty text with country-specific rates separated by semicolons |
| **Tariff Paragraph** | 25 | Including Revenue Act references |

### Other Editions
Define columns dynamically based on the source document's table headers. At minimum extract: commodity number, description, unit of quantity, rate(s) of duty, and tariff paragraph.

---

## Commodity Number Formats

### 1950 Edition
- Already in `XXXX XXX` format with a space

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SU-OSPO/tariff-claude-skill](https://github.com/SU-OSPO/tariff-claude-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
