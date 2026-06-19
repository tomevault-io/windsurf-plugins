---
trigger: always_on
description: Use when the user wants to summarize a lecture PDF, course slides, or课件 into复习资料, 考点总结, 公式/概念速查表, or an exam-oriented cheat sheet PDF with page-cited key points, formulas, concepts, examples, and study guidance.
---


# Summarize Slides

Turn a long lecture PDF into a concise, exam-oriented LaTeX cheat sheet and compiled PDF with accurate page references, bilingual terminology, and only the diagrams that materially improve understanding.

**Core principle:** treat slide summarization as a controller workflow for long documents: read globally first, segment by lecture or topic, dispatch focused readers for each segment, merge into a single cheat-sheet draft, then run a fresh coverage verifier before building final artifacts.

## Non-Negotiables

- **REQUIRED COMPANION SKILL:** use `pdf` for PDF intake and reading. This workflow is not optional.
- **NO OTHER SKILLS:** when this skill applies, use only `summarize-slides` plus `pdf`. Do **not** invoke `dispatching-parallel-agents` or any other additional skill. Any task-splitting, reader dispatch, verifier separation, or parallelization logic needed for this workflow is already defined inside this skill.
- Check that locally installed `xelatex` is available before promising the default required compiled PDF output.
- Default deliverables are `.tex` and compiled `.pdf`, with the PDF treated as the required final artifact unless the user explicitly asks not to produce it, and the default compiled PDF path requires `xelatex`.
- When this skill applies, a chat-only or inline summary is **not** a successful final deliverable unless the user explicitly asks for summary-only output or explicitly opts out of file creation.
- If the user gives parsed text, OCR text, extracted images, or page-by-page content originating from a lecture PDF instead of the raw PDF file, treat it as the same slide-summarization task and still produce the document artifacts by default.
- By default, place all generated artifacts in a single folder named `Summary - <pdf-stem>` inside the same directory that contains the source PDF. Do not scatter `.tex`, `.pdf`, helper assets, or intermediate files elsewhere.
- This containment rule is strict: create the output folder before any extraction, note-taking, OCR, or build step, and write every generated file into that folder from the start.
- Extracted text files are included in this rule. Files such as `<pdf-stem>_full.txt`, `<pdf-stem>_p1-5.txt`, OCR dumps, page-range extracts, verifier notes saved to disk, temporary LaTeX snippets, and any other intermediate `.txt`, `.md`, `.json`, `.tex`, image, or helper outputs must live inside `Summary - <pdf-stem>` only.
- Never place generated files next to the source PDF unless the user explicitly asked for that location. The source PDF's directory is not a scratch space.
- Here, `same directory that contains the source PDF` means the output folder sits alongside the PDF file itself. Example: if the source PDF is `.../lecture/foo.pdf`, the default output folder is `.../lecture/Summary - foo/`.
- The default writing style is Chinese-first.
- Important technical concepts, terms, and named distributions/theorems must include the original English in parentheses.
- Each summarized point should include the source slide page number or a tight page range by default. Omit citations only if an extraction failure is explicitly reported.
- The output style is `cheat sheet` in the sense of coverage, brevity, clarity, and fast lookup. It does **not** mean forced ultra-dense layout, automatic two-column formatting, or unreadable compression.
- Default layout is normal single-column reading unless the user explicitly asks for another layout.
- Do not produce a generic chapter summary when the user asked for exam points.
- Include formulas, important concepts, representative examples, and problem-solving techniques where they matter.
- Do not silently drop later sections of a long PDF because of context or time pressure.
- If extraction quality is too poor to preserve formulas, symbols, or page mapping, stop and ask rather than paraphrasing uncertain mathematics.
- If an important concept genuinely needs a figure to explain it clearly, include a LaTeX-drawn explanatory figure in the final document.
- Do not use screenshots or copied slide images as a substitute for a requested or necessary explanatory figure unless the user explicitly permits that fallback.

## Skill Boundary

- Allowed skills for this workflow: `summarize-slides` and `pdf` only.
- Forbidden: all other skills, even if they appear relevant to planning, dispatch, verification, or writing.
- Task or subagent mechanisms may still be used when the platform supports them, but that is an execution detail and does **not** justify loading another skill.

## Required Outcome

The default successful outcome for this skill is:

- a saved LaTeX source file named `Summary - <pdf-stem>.tex`
- a compiled PDF file named `Summary - <pdf-stem>.pdf`
- all generated files grouped in one output folder named `Summary - <pdf-stem>` inside the same directory as the source PDF unless the user explicitly requested another location
- no generated intermediate, extracted-text, OCR, notes, or helper files left outside that output folder
- a final response that reports the artifact paths and any blockers encountered


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Li-Baichuan-James/summarize-slides-skill](https://github.com/Li-Baichuan-James/summarize-slides-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
