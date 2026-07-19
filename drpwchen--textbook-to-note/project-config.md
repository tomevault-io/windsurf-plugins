---
trigger: always_on
description: You are helping your user set up `textbook-to-note`: a local-first pipeline
---

# AGENTS.md — Instructions for your AI coding agent

You are helping your user set up `textbook-to-note`: a local-first pipeline
that converts their own PDF/EPUB textbooks into searchable markdown, then
into structured, fully-cited notes in their personal knowledge vault
(Obsidian, Logseq, or a plain markdown folder). This repository is designed
to be deployed **by you**, an AI coding agent, working directly with the
user rather than requiring them to hand-run every script themselves.

Read this file fully before doing anything. Then work through the steps
below in order, checking in with the user at the marked decision points.

## What you're setting up

```
converter/    — PDF/EPUB → markdown conversion (0 LLM tokens)
figures/      — on-demand figure extraction with QC gating
skills/       — two Claude Code skill definitions (drop-in to ~/.claude/skills/)
workflows/    — the note-writing workflow specification
templates/    — real production note templates (zh-TW + English) for Step 1.1's topic-type table
docs/         — architecture + OCR-ladder reference docs
examples/     — one example output note showing the target format
shared/       — shared config (paths, env var names)
requirements.txt
```

Read `docs/architecture.md` first for the full picture, and
`docs/ocr-ladder.md` if you'll be doing any OCR-heavy conversions.

## Step 1: Understand the user's situation

Ask, or infer from context:
- Where do their textbook PDFs/EPUBs already live? (a folder, a cloud-synced
  drive, etc.)
- What notes tool do they use, and where is their vault/notes folder?
- Do they want the optional semantic search index (LanceDB + a local
  embedding model via ollama), or is grep-only fine for their corpus size?
  Semantic search pays off once there are more than a handful of books;
  for a small personal library, skip it initially and add later.
- Do they have a GPU available locally? This affects whether the OCR ladder
  (`docs/ocr-ladder.md`) can use a GPU-accelerated engine or should fall
  back to CPU-only options. See "Choosing your hardware tier" in
  `docs/ocr-ladder.md` for the tier table (CPU-only / Apple Silicon 8GB /
  Apple Silicon 16GB+ / NVIDIA 8GB / NVIDIA 16GB+) before recommending any
  OCR/vision-QC/embedding stack.

## Step 2: Install dependencies

```bash
pip install -r requirements.txt
```

`requirements.txt` covers PDF parsing, EPUB conversion (needs `pandoc` on
`PATH` separately — check with `pandoc --version` and prompt the user to
install it if missing), and the optional semantic-search stack. If the user
declined semantic search in Step 1, you can skip installing that subset.

## Step 3: Configure paths

Configuration is environment-variable driven with repo-relative defaults —
`shared/config.py` documents every variable and works with zero setup:
drop books in `./books`, get markdown in `./output`.

Set env vars (in the shell, or a `.env` you source) only where the defaults
don't fit:
- `BOOKS_DIR` — where the user's PDFs/EPUBs live (default `./books`)
- `OUTPUT_DIR` — where converted markdown goes (default `./output`; keep it
  outside the notes vault — this corpus is for your reference, not for the
  user to read directly)
- Optional OCR fallback: `SURYA_VENV_PY` + `SURYA_ADAPTER` (see
  `docs/ocr-ladder.md`)
- Optional semantic search: `INDEXER_SCRIPT` + `VAULT_SEARCH_DIR`
- Figure output/cache locations: env-driven constants documented at the top
  of `figures/figure_qc_gate.py` (default `./output/figures`, inside the
  user's vault attachments folder if they want embeds to resolve)

Run `python shared/config.py` to print the resolved configuration and
confirm it before converting anything. Never hardcode paths into scripts or
skill files — always go through `shared/config.py` or env vars, so the repo
stays portable across machines and users.

## Step 4: Convert a first book (smoke test)

Pick one book the user cares about and convert it end to end, narrating what
you're doing:

```bash
python converter/convert.py "path/to/one.pdf" --book-label "Author Title — Ch.1"
```

Check the output markdown for:
- Readable, non-garbled text
- `<!-- page N -->` markers present
- Any `<!-- REF: Fig. X.Y → ... -->` markers where the source mentions
  figures

If the text looks garbled or mostly empty, read `docs/ocr-ladder.md` and
re-run with the OCR-forcing flag (`--force-surya`, requires the optional
OCR engine from `docs/ocr-ladder.md`) rather than assuming the conversion is
broken — most garbled output on a first try is a silent fitz failure that
the OCR ladder is built to catch.

Once the smoke test looks right, convert the rest of the user's priority
books:

```bash
python converter/convert.py --batch-dir "path/to/their/textbook/folder"
```

This can take a while for a large library — run it as a background job and
report progress, don't block the conversation on it. Don't run two batch
conversions against the same `OUTPUT_DIR` concurrently — progress tracking
is last-writer-wins, so overlapping runs will corrupt each other's progress
state.

## Step 5: (Optional) Build the semantic index

Only if the user opted in during Step 1:

```bash
python converter/post_convert.py --index
```

Requires the local embedding model to be running (e.g. `ollama pull

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drpwchen/textbook-to-note](https://github.com/drpwchen/textbook-to-note) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
