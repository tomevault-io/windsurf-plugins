---
trigger: always_on
description: Build a modern desktop GUI application in Python that sorts PDF files into
---

Build a modern desktop GUI application in Python that sorts PDF files into
AI-generated categories based on their content, using a local Ollama LLM.
No external API calls, no API keys, fully offline.

## Framework and packaging

- GUI framework: customtkinter (modern, light/dark mode aware, pip installable).
- Use pyproject.toml with project name `pdf-sorter`, requiring Python >= 3.10.
- Dependencies: customtkinter, pymupdf, ollama, rapidfuzz, Pillow.
- Entry point: python -m pdf_sorter  (launches the GUI window).
- Also provide a platform-specific launcher in the README:
    macOS/Linux: a one-line shell script  `run.sh`  containing  `python -m pdf_sorter`
    Windows:     a  `run.bat`  file doing the same.
- Package structure:
    pdf_sorter/
        __init__.py
        __main__.py      # calls app.run()
        app.py           # main App class, window setup, view switching
        views/
            __init__.py
            setup_view.py     # configuration panel (Step 1)
            progress_view.py  # live progress panel (Step 2)
            preview_view.py   # categorization results table (Step 3)
            done_view.py      # final summary (Step 4)
        backend/
            __init__.py
            extract.py        # PDF text extraction
            categorize.py     # Ollama LLM inference
            normalize.py      # label normalization with rapidfuzz
            files.py          # file moving and collision handling
            log.py            # sort_log.txt writing
        worker.py             # background thread orchestration

## Application flow

The app is a single window (min 720 x 600 px, resizable) that steps through four
views. The active view fills the window. Navigation between views is handled by
app.py's show_view(view_name) method.

---

### View 1 — Setup (setup_view.py)

This is what the user sees on launch. All configuration happens here.

Layout (top to bottom):

  [App title: "PDF Sorter"]
  [Subtitle: "Sort your PDFs into folders using a local AI"]

  INPUT FOLDER
  [Text field showing selected path]  [Browse... button]
  (opens a native folder picker dialog via tkinter.filedialog.askdirectory)

  OUTPUT FOLDER
  [Text field showing selected path, pre-filled with <input_dir>/sorted/]  [Browse... button]
  Hint text below: "Category subfolders will be created here."

  OLLAMA MODEL
  [Dropdown (CTkComboBox) listing available local models]  [Refresh button ↺]
  The dropdown is populated by calling `ollama list` via the ollama Python package
  on view load and on Refresh. If no models are found, show:
  "No models found. Run: ollama pull llama3.2:3b" in red below the dropdown.
  If Ollama is not running, show:
  "Ollama is not running. Start it with: ollama serve" in red.
  Recommended models hint (static, always visible):
  "Recommended: llama3.2:3b (8 GB RAM) · llama3.1:8b (16 GB RAM)"

  CATEGORY HINTS  (optional)
  Label: "Preferred categories (optional)"
  [Text field]  [+ Add button]
  Below the field: a wrapping row of removable chip/tag widgets for each added hint.
  Each chip has an × button to remove it.
  Hint text: "Guide the AI toward these categories. Leave empty for fully automatic."

  ADVANCED OPTIONS  [collapsible section, collapsed by default]
    Parallel extraction workers: [Slider 1–8, default 4, integer label beside it]
    Dry run (preview only):      [Toggle switch]

  [Start Sorting  →  button, full-width, accent color, disabled until input_dir is set]

On clicking Start Sorting:
  1. Validate that input_dir exists and contains at least one PDF. If not, show an
     inline error below the input field.
  2. Validate Ollama is running and selected model is available. If not, show inline
     error below the model dropdown.
  3. If output_dir already contains sort_log.txt, show a modal dialog:
     "This output folder was used before. Continue anyway?" [Yes] [Cancel]
  4. If all checks pass, switch to View 2.

---

### View 2 — Processing (progress_view.py)

Shown while extraction and categorization are running. The user cannot go back.
A Cancel button aborts gracefully (finishes the current file, then stops).

Layout:

  [Title: "Sorting your PDFs..."]

  Extraction
  [Progress bar]  "34 / 100 files"
  [Current file label: "Extracting: quarterly_report.pdf"]

  Categorization
  [Progress bar]  "12 / 100 files"
  [Current file label: "Categorizing: quarterly_report.pdf"]  [model name badge]

  [Scrollable log area showing one line per completed file:]
    ✓  invoice_jan.pdf       →  Invoice
    ✓  gpt4_paper.pdf        →  Research Paper
    ✗  scan001.pdf           →  Uncategorized  (no extractable text)

  [Cancel button, right-aligned]

The log area uses monospaced font. Green ✓ for successfully categorized, orange ✗
for Uncategorized. Each new line is appended as it completes; the area auto-scrolls.

All processing runs in a background thread (worker.py) and posts updates to the GUI
via a thread-safe queue that the GUI polls with app.after(50, poll_queue).

On completion (or after cancel): switch to View 3.

---

### View 3 — Preview (preview_view.py)

Shows the categorization results before any files are moved.
If dry-run mode is on, the "Move Files" button is hidden and a "Dry Run" badge is
shown instead.

Layout:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [erxuanli/file-organizer](https://github.com/erxuanli/file-organizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
