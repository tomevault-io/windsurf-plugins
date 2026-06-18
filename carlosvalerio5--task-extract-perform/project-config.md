---
trigger: always_on
description: Processes PDF slide presentations by extracting deck content and end-of-deck activity instructions into paired markdown files, optionally emits a per-activity Python script when instructions call for runnable examples or practice, produces LaTeX write-ups per activity, aggregates all activities in a phase into one TeX document, compiles to PDF, zips the final PDF together with generated activity scripts, and treats that archive plus the full phase tree as the deliverable. Use when working with p
---


# PDF phase presentations → markdown pairs → phase activity PDF

## Scope

- **Input**: One or more PDF presentations that belong to **phases** (phase identifiable from the file name).
- **Per presentation**: Two markdown files — **info** (deck context) and **activity** (instructions from the end of the deck).
- **Per presentation (optional)**: One **Python script** when instructions clearly benefit from a runnable example, simulation, data prep, or hands-on practice; omit when not applicable.
- **Per presentation**: One LaTeX file that is the **completed activity** (answer / deliverable).
- **Per phase**: One aggregated TeX file that includes all activity results for that phase, **compiled to PDF**, a **zip archive** containing that PDF and **all** `{base}_activity_script.py` files for the phase (see final workflow step), plus **all artifacts for that phase** including every markdown pair, each `_activity.tex`, and each generated script on disk (see layout below).

## PDF text vs scanned slides

- If the PDF has selectable text, extract with a text-based library (e.g. `pdfplumber` or PyMuPDF).
- If slides are image-only or OCR is needed, run OCR first (e.g. Tesseract + `pdf2image`, or any workflow that yields markdown or plain text per slide), then continue this pipeline using that extracted text as the source.

## Phase grouping (file names)

Infer **phase** from a consistent naming convention. Examples the user or project should adopt:

- `phase1_01_intro.pdf`, `phase1_02_models.pdf` → phase `phase1`
- `P2-SlideDeck-TopicA.pdf` → phase `P2`

If names are ambiguous, ask the user once for the rule (regex or prefix/split pattern), then apply it to all files in the batch.

## Output layout (recommended)

Use a dedicated folder per phase so aggregation is obvious:

```text
output/
  phase1/
    decks/
      01_intro_info.md
      01_intro_activity.md
      01_intro_activity.tex         # completed activity (LaTeX fragment)
      01_intro_activity_script.py   # optional: only if applicable (see below)
    phase1_activities.tex           # \input{} or \include{} of each *_activity.tex
    phase1_activities.pdf           # compiled output
    phase1_deliverable.zip          # PDF + scripts/… (see workflow step 12)
```

Adjust names to match the user’s naming scheme; keep the **pairing rule** fixed: one base name → `*_info.md`, `*_activity.md`, `*_activity.tex`. **Optional script** name: `{base}_activity_script.py` (same `{base}` as the markdown pair, e.g. `01_intro_activity_script.py`).

**Phase deliverable:** Treat the whole `output/<phase>/` tree (PDF, master `.tex`, `decks/*.md`, `decks/*_activity.tex`, **any** `decks/*_activity_script.py`, and **`{phase}_deliverable.zip`**) as the consolidated output for the phase. The zip is the handoff bundle for **compiled PDF + scripts**; other files remain for editing and rebuilds.

## Workflow

Follow these steps in order.

1. **Read the presentation** (PDF). Determine phase from the file name.
2. **Extract relevant information** into an `.md` file (concepts, definitions, examples from the deck — not the activity block yet). This is the **info** markdown.
3. **Read the activity instructions** at the end of the presentation (last slides or labeled section).
4. **Extract the instructions** into a **second** `.md` file (activity-only, verbatim or clearly structured).
5. **Each presentation** must yield a **pair** of markdown files: **info** + **activity instructions**.
6. **Optional Python script for this activity** (after the activity `.md` exists):
   - Decide whether a **runnable Python script** would materially improve results: e.g. numerical experiment, simulation, data/file processing, plotting, algorithm demo, scaffold for exercises that ask for code, or “try it yourself” practicals aligned with the instructions.
   - **Clearly applicable:** Write `{base}_activity_script.py` next to the other deck artifacts. Keep it focused on the activity (docstring referencing the activity instructions, minimal deps, `if __name__ == "__main__":` entry when useful). Use only standard library unless the user or project already specifies dependencies.
   - **Clearly not applicable** (pure reflection/essay, untyped discussion, multiple-choice-only, no computational or practical hook): **do not** create a script.
   - **Unclear:** **Ask the user** whether they want a script for this activity and what it should demonstrate or automate; only create it after they confirm (or decline).
7. **Create a final `.tex` file** with the **result of the activity** (the completed work: answers, reflection, design, etc.), using the activity `.md` as the spec for what to deliver. If a companion `{base}_activity_script.py` exists, **reference it** in the write-up (filename and one-line purpose); see [reference.md](reference.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlosValerio5/task-extract-perform](https://github.com/carlosValerio5/task-extract-perform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
