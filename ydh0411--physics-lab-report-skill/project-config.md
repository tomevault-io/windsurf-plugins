---
trigger: always_on
description: Use when generating, auditing, or revising UESTC Physics Experiments I/II lab reports, prelab预习报告, postlab实验报告, or related LaTeX templates/skills for 大物实验报告.
---


# UESTC Physics Lab Report Generator

## Overview

Generate or audit prelab (预习报告) and postlab (实验报告) for UESTC Physics Experiments I/II using XeLaTeX. Two distinct templates share core infrastructure: cover page via `\includepdf`, score boxes, diagonal watermark, and real-data tables. The templates are designed for any UESTC student, but the rules must always be checked against the current school template, lab manual, scanned data, and reference reports.

## Work Modes

This skill supports three modes of operation:

### First-Time Setup

Run once at the start of the semester. The student provides stable information that is reused for every report:

1. Upload prelab and postlab cover template PDFs
2. Confirm student profile (name, student ID, email, college, major, class, instructor, TA) — see `config/student_profile.example.yaml`
3. Set PDF naming format for submission — see `config/naming.example.yaml`
4. Confirm course/textbook version

After setup, config files and templates are reused automatically. To change stable info later, use Update Mode.

### Regular Generation

What the student does for each report:

1. Choose **prelab** or **postlab**
2. Provide the **full experiment title** (e.g. "Polarization of Light")
3. Provide the **lab number** (for cover page, filename, and submission order only)
4. **Manually input experimental data** (recommended; see Data Input below)
5. Optionally upload a teacher-specific formula template or data-processing sheet
6. Optionally upload the completed scanned data sheet for the appendix
7. Skill generates `.tex` source and compiles to final PDF

### Update Mode

Used when stable information changes:

- Update cover template PDFs
- Update student profile (new instructor, TA, etc.)
- Update PDF naming rules
- Update course/textbook version
- Update experiment formula templates or reference files

## Report Types at a Glance

| | Prelab | Postlab |
|---|---|---|
| Cover PDF | `01-Template for Prelab work-2026*.pdf` | `02-Template for lab report-2026*.pdf` |
| Cover title | "Prelab Report" | "Lab Report" |
| Page style | `empty` | `fancy`, centered page numbers |
| Watermark | "Physics Lab 2026" (singular) | "Physics Labs 2026" (plural) |
| Sections | One: "Answers to Questions (20 points)" with overlay score box | Four: Abstract (5), Calculations/Results/Comments (15), Conclusions (10), Questions (10) |
| Multi-page | Repeat background + foreground per page; continue `enumi` counter | Natural text flow |
| Answer format | `(\arabic*)` enumeration, no left margin | `\item \textbf{Title}` with full paragraphs |
| Header on page 2+ | Absolute-position overlay with student info + score box | None (info is on cover only) |

## Compilation

**Must use XeLaTeX** (not pdfLaTeX). Two passes required. Work in a single directory containing all `.tex`, cover PDFs, data PDFs, and figure PDFs.

Use the provided build script for a one-command PDF-first workflow:

```bash
bash scripts/build_report.sh file.tex "final_submission_name.pdf"
```

Or compile manually:

```bash
xelatex -interaction=nonstopmode file.tex
xelatex -interaction=nonstopmode file.tex
```

The build script also checks XeLaTeX availability, cleans auxiliary files, and renames the output PDF. Overleaf is a fallback option if XeLaTeX is not installed locally.

## Personal Info and Inputs

After first-time setup, student identity comes from `config/student_profile.yaml`. The skill loads it automatically; the student does not need to re-enter it for each report.

For one-off use without config files, prelab pages after the cover need overlay variables:

```latex
\newcommand{\StudentNumber}{STUDENTID}
\newcommand{\StudentEmail}{student@example.com}
\newcommand{\ReportDate}{2026.X.X}        % dots format for overlay
\newcommand{\CoverPDF}{01-Template for Prelab work-2026.pdf}
```

Postlab usually gets student identity from the filled cover PDF itself. Its working variables are normally:

```latex
\newcommand{\CoverPDF}{02-Template for lab report-2026.pdf}
\newcommand{\DataPDF}{scanned-data.pdf}
% optional figure PDFs generated from real data
```

The cover page date (filled in the template PDF itself) may use `/` format (e.g., `2026/5/12`). The overlay date on prelab page 2 uses `.` format (e.g., `2026.5.12`). These can differ by a day (submission date vs. writing date).

## Core Principles

### 1. Real Data Only

All numbers must trace to scanned data sheets, instrument specs, lab-manual values, or textbook values. Extract text with `pdftotext`/Python when possible. If a PDF is scanned and text extraction is empty, render pages with `pdftoppm` and use OCR or manual visual transcription. Never fabricate data; when data is unclear, say so or transcribe cautiously from the scan.

### 2. Cover via includepdf

```latex
\usepackage{pdfpages}
\includepdf[pages=1,pagecommand={\thispagestyle{empty}}]{\CoverPDF}
\setcounter{page}{1}
```

Never draw the cover in LaTeX.

### 3. Score Boxes

Postlab: `\labsection{Title}{(points)}` renders a left score box + heading.
Prelab: score box drawn in foreground overlay with `\rule` primitives.

### 4. Data Table Format

```latex
\datatable{DATA TABLE X-X}{(Purpose: ...)}
\begin{table}[H]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ydh0411/physics-lab-report-skill](https://github.com/ydh0411/physics-lab-report-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
