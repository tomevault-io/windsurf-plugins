---
trigger: always_on
description: Comprehensive quality control for U.S. patent application filing documents. Use when the user asks to QC, check, validate, or review patent filing documents before submission to the USPTO. Performs 70+ automated and manual checks across specification, drawings, ADS, declaration, assignment, and power of attorney documents. Detects cross-document inconsistencies, formatting issues, missing required fields, and common errors.
---


# Patent Filing QC

## Overview

This skill provides comprehensive quality control for U.S. patent application filing documents immediately before filing with the USPTO. It systematically checks for internal document errors, cross-document inconsistencies, USPTO compliance issues, and common filing mistakes across all required and optional filing documents.

## When to Use This Skill

Trigger this skill when the user requests any of the following:
- "QC this patent application"
- "Check these filing documents"
- "Validate this patent package"
- "Review this application before filing"
- Any request to quality check, validate, or review patent filing documents

## Workflow

### Step 1: Identify the Documents Folder

Confirm the folder path containing all filing documents. The folder should typically contain:
- Specification PDF (required)
- Drawings PDF (required)
- Application Data Sheet / ADS (required)
- Declaration (eligible for missing-parts under 37 CFR §1.53(f) — see Step 8)
- Assignment (optional but common)
- Power of Attorney (optional)
- `inventors.txt`, `inventors.json`, or `*.eml` (optional — authoritative inventor list)

**Filenames don't matter — the script identifies documents by their *content*, not their names.** A specification named `Application.pdf`, a declaration named `Formals.pdf`, or any other naming convention works as long as the content of the file is recognizable. XFA forms (any PTO/AIA/* form, not just the ADS) are classified by inspecting their embedded XML rather than assuming form type from the filename.

**Format-wise, the spec can be `.docx` or `.pdf`.** The USPTO accepts the specification in Word format, and so does this tool — the classifier reads .docx the same way it reads PDFs and routes it to the Specification slot when its content matches. Other documents (declaration, ADS, drawings, assignment, POA) should be PDFs.

**Note on the ADS:** The USPTO web-fillable ADS (PTO/AIA/14) is an XFA form. The script reads the form's embedded XFA datasets stream directly, so no Adobe Acrobat Pro flattening is required. If the ADS extraction succeeds, the console will show `✅ XFA extraction successful` — every ADS field needed for cross-document checks is then available structured.

### Step 2: Install Dependencies

The user does not need to install anything by hand. If the script fails or warns about a missing dependency, you should install it for them automatically (asking permission once when prompted) and re-run.

Required:
- **PyPDF2** — `pip install PyPDF2 --break-system-packages`. Used for XFA stream extraction and AcroForm inspection. If you see `ModuleNotFoundError: No module named 'PyPDF2'`, install it before re-running.
- **pdfplumber** — `pip install pdfplumber --break-system-packages`. Used as the primary text extractor for spec/declaration/assignment/POA. Preserves paragraph structure that PyPDF2 strips, which the section-detection and claim-parsing checks rely on. Without it, the script falls back to PyPDF2 and many spec-content checks will produce false positives.

For Word (.docx) specifications (the USPTO accepts the spec in .docx; everything else stays PDF):
- **python-docx** — `pip install python-docx --break-system-packages`. Only required if a `.docx` file is present in the folder. The script will print an install hint if it encounters a `.docx` without python-docx installed.

The report is HTML with embedded CSS (no pandoc, no LaTeX, no weasyprint required). If the user wants a PDF copy, they open the HTML in any browser and use File → Print → Save as PDF.

Optional (only for image-based / scanned PDFs that aren't text-searchable; **not** needed for the USPTO XFA-based ADS):
- `pip install pytesseract pdf2image --break-system-packages` + `brew install tesseract poppler`

### Step 3: Run the QC Script

Execute the comprehensive QC check:

```bash
python3 /path/to/skill/scripts/qc_patent_filing.py <folder-path>
```

Optional: Specify output directory for reports:
```bash
python3 /path/to/skill/scripts/qc_patent_filing.py <folder-path> --output-dir <output-path>
```

Optional: Lightweight (filing-identity-only) mode — skips drafting-quality
checks (antecedent basis, terminology consistency, abstract length, optional
USPTO formatting, etc.) and reports only the checks that catch a wrong or
mismatched file at filing time. Use when the specification is already
drafting-reviewed and the remaining risk is attaching the wrong file:
```bash
python3 /path/to/skill/scripts/qc_patent_filing.py <folder-path> --lightweight
```

The script will:
1. Automatically detect all filing documents in the folder
2. Extract text from PDFs
3. Run all 70+ quality control checks
4. Generate one self-contained HTML report file in the output folder

### Step 4: Ensure the Report Is Generated

The script writes one report file:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeonardHope/Claude-Skill-for-Patent-Filing-Quality-Control](https://github.com/LeonardHope/Claude-Skill-for-Patent-Filing-Quality-Control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
