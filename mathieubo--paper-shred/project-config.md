---
trigger: always_on
description: |
---


# /paper-shred — PDF → structured folder of markdown

Shred a scientific PDF into individually addressable components, following the
"biomedical literature as a filesystem" principle (https://gxl.ai/blog/biomedical-literature-as-a-filesystem).
Each paper becomes a folder where sections, figures, tables, and references are
separate files an agent can grep/read directly without ingesting the whole doc.

**Two extractors run in parallel:**
- **marker-pdf** — primary source for body text. Best at preserving `<sup>` citation wrappers, bold/italic emphasis, and scientific notation (e.g. `Atg7<sup>flox/flox</sup>`).
- **docling** — auxiliary source for document-level metadata. Best at detecting the document title, providing a clean heading scaffold (uniform H2s instead of marker's inconsistent levels), and counting figures/tables.

Claude reconciles them: body content from marker, title + structural scaffold from docling.

**Argument**: `$ARGUMENTS` should be a path to a PDF. If empty or invalid,
tell the user: `Usage: /paper-shred /path/to/document.pdf [output_parent_dir]`.

If `$ARGUMENTS` is a **directory**, switch to batch mode (see below) instead of
running the per-PDF Claude flow — for >5 PDFs the per-paper restructuring is
impractical in a single session.

---

## Batch mode (directories of >5 PDFs)

```
python ${CLAUDE_SKILL_DIR}/bin/shred_batch.py <directory> [--require-caption]
```

Walks the directory for `*.pdf` (skipping symlinks and anything under `_raw/`),
runs the same extract → clean → audit → split → write → post-pass pipeline per
file using deterministic heuristics — no Claude in the loop. Idempotent: a
folder with `meta.json + README.md` is skipped on re-run; the extract.sh cache
makes "delete user-facing output, re-shred" finish in seconds. Per-paper status
is appended to `<directory>/_shred_log.jsonl`. Expect 5–10 min/PDF on CPU
(50 PDFs ≈ 5–8 hours wall time). Use `--require-caption` on Cell/Nature
collections to drop figure panels without an anchored caption.

The single-PDF Claude-assisted flow below stays the right choice when you want
careful structural decisions (grants, theses, reviews with unusual layouts).

---

## Step 1 — Validate and resolve paths

Parse `$ARGUMENTS`:
- First token: PDF path (required, must end in `.pdf` and exist).
- Second token (optional): parent directory for the output folder. Default:
  the directory containing the PDF.

Compute:
- `STEM` = PDF basename without `.pdf`, sanitized (replace whitespace/punct with `_`)
- `OUT_DIR` = `<output_parent_dir>/<STEM>`
- `WORK` = `<OUT_DIR>/_raw` (working dir for marker output)

If `OUT_DIR` already exists with a `README.md`, ask the user whether to
overwrite, append a suffix (`_v2`), or abort. Don't silently overwrite.

---

## Step 2 — Run extractors (marker-pdf + docling, parallel)

```bash
bash ${CLAUDE_SKILL_DIR}/bin/extract.sh "<PDF>" "<WORK>"
```

This launches both extractors in background and waits for both. Stdout emits
`key=value` lines: `marker_md`, `marker_meta`, `marker_dir`, `docling_md`,
`docling_meta`, `docling_pics`, `docling_title`, `stem`, `marker_venv`,
`docling_venv`. Capture all of them.

If marker fails the script aborts. If docling fails (or its venv is missing),
the script emits a warning and continues with marker-only output — `docling_md`
and friends will be empty strings; downstream logic must handle that.

**Runtime expectations** (CPU, 5-page document):
- First run: 5–10 minutes (marker downloads ~2 GB of surya models; docling
  downloads ~50 MB of RapidOCR + layout models).
- Cached: marker ~1–7 minutes depending on page count, docling ~1–3 minutes,
  running in parallel — wall time dominated by marker.
- A GPU drops marker to ~0.2 s/page; not required.

---

## Step 3 — Mechanical pre-pass

```bash
"$venv/bin/python" ${CLAUDE_SKILL_DIR}/bin/mechanical_clean.py \
    "$marker_md" "$WORK/cleaned.md"
```

This strips Zotero/google-doc tracking URLs, drops `{N}-----` page-break
separators, replaces `<br>` tags, and normalizes whitespace. It does **not**
attempt heading promotion or reference splitting — that's your job in Step 4.

Note the stderr summary line: chars, image count, table count, and citation
counts broken down by style (`sup`, `bracket`, `authoryear`) plus the dominant
style. Use the dominant-style count to populate `meta.json.n_citations_inline`
and record the style in `meta.json.citation_style`. The author-year counter is
loose by design (it also matches bibliography entries) — when the dominant
style is `authoryear` the count is roughly "inline citations + bibliography
entries", so subtract the reference-list size if you need a tight inline count.

---

## Step 4 — Read and plan the structure

Read `$WORK/cleaned.md`. Then plan the folder layout:

### 4a. Identify document type

From cues in the text, classify as one of: `paper`, `grant`, `review`,
`preprint`, `thesis`, `protocol`, `other`. Heuristics:
- `paper`: Abstract + Introduction + Results + Discussion + Methods/Materials
- `grant`: Specific Aims, Vision/Approach, Significance, Innovation, Objectives,
  Research Strategy, Budget, Biosketch, Letters of Support
- `review`: lots of citations, no Methods/Results, narrative section names
- `preprint`: paper-shaped, with bioRxiv/medRxiv/arXiv markers
- `thesis`: Chapters 1..N, Acknowledgements, Declaration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MathieuBo/paper-shred](https://github.com/MathieuBo/paper-shred) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
