---
trigger: always_on
description: >-
---


# Paper Reader

Use this skill to read an academic PDF in either full mode or fast-report mode.
The goal is not to translate every sentence. Reconstruct the authors' reasoning,
identify the core problem, explain why prior approaches are insufficient, and
extract the key insights.

- **Full mode**: produce two Chinese Markdown files: one concise translated
  logic reconstruction and one research-level analysis.
- **Fast-report mode**: produce one Chinese Markdown file focused on task
  definition, method details, experiments, first-principles reconstruction, and
  literature positioning.

## Inputs and Outputs

- If the user did not provide a PDF path, ask for one concise question.
- Resolve the PDF path before working. Put all outputs in the same directory as
  the PDF.
- Use fast-report mode when the user asks for "fast", "quick", "快速",
  "速读", "快读", "快速报告", "fast report", or requests a single compact
  paper report. Otherwise use full mode.
- In full mode, create exactly these two files:
  - `<original_filename>_translation.md`
  - `<original_filename>_analysis.md`
- In fast-report mode, create exactly this one file:
  - `<original_filename>_fastreport.md`
- All output files must be written entirely in Chinese. Section titles, table
  headers, explanations, and commentary must be Chinese. Mathematical formulas,
  variable names, method names, benchmark names, model names, code identifiers,
  and paper-internal identifiers may remain in their original form.

## Evidence Rules

1. Do not hallucinate. Every factual claim must be traceable to the paper.
2. Separate paper facts from your interpretations.
3. When evidence is insufficient, write: `论文未提供足够证据。`
4. Prefer citing the paper location when possible: section, page, figure, table,
   appendix, equation, or algorithm.
5. Preserve uncertainty. Do not convert author speculation into established
   fact.
6. Do not use external sources unless the user asks, the paper explicitly relies
   on an external artifact, or the task cannot be completed without checking a
   referenced repository/document. If external sources are used, clearly label
   them separately from paper evidence.

## Markdown and LaTeX Rules

1. Write math in Markdown-compatible LaTeX. Use `$...$` for inline math and
   `$$...$$` for display math. Do not use `\(...\)` or `\[...\]` delimiters.
2. Preserve LaTeX commands with literal backslashes in the final Markdown file.
   For example, write `$4.22 \times 10^{20}$`, not `\(4.22\times10^{20}\)`,
   `4.22 times 10^20`, or `4.22 x 10^20`.
3. Reconstruct malformed extracted formulas into valid LaTeX when the PDF text
   extraction drops symbols, spacing, superscripts, subscripts, or Greek letters.
   If the formula cannot be recovered from the paper, write
   `论文未提供足够证据。`
4. Keep formulas in math mode even inside Chinese sentences and tables. In
   Markdown tables, avoid raw `|` inside formulas; use `\mid`, `\vert`,
   `\lVert...\rVert`, or move the formula outside the table.
5. Do not wrap formulas in code backticks unless explicitly discussing source
   code. Do not translate LaTeX operators or variables into Chinese inside math.

## Reading Workflow

1. Extract readable text from the PDF. Prefer structured extraction when
   available, such as `pdftotext -layout`; if extraction is poor, try another
   available local tool or inspect relevant pages visually.
2. Build an internal evidence map before writing:
   - title, authors, venue/date if present
   - abstract and stated contributions
   - problem setup and assumptions
   - method components and information flow
   - datasets, baselines, metrics, tables, figures, ablations, appendices
   - stated limitations and unstated failure modes
3. Read figures, tables, equations, algorithms, and appendices, but extract them
   at different granularity:
   - Core methods, algorithms, definitions, equations, architectural diagrams,
     main results, key ablations, and central claims must be preserved.
   - Repetitive background, rhetorical setup, duplicated claims, routine dataset
     descriptions, and boilerplate experimental details may be merged or omitted.
   - Appendices and supplementary experiments should usually be summarized by
     their purpose, evidence value, and conclusion. Only expand appendix content
     when it contains essential proofs, implementation details needed to
     understand the method, or results that materially change the interpretation.
   - Reproduce a table only when exact numbers are central to the paper's
     argument. Otherwise summarize the trend and cite the table/figure location.
4. For long papers, process in chunks but keep a global outline so later
   sections do not contradict earlier evidence.
5. In full mode, generate the translation first, then the analysis. Do not stop
   after only one file. In fast-report mode, generate only the fast report.

## Fast Report File

Create `<original_filename>_fastreport.md`.

This file is a compact Chinese research report for quickly understanding a paper
without producing the full translation and analysis pair. It must still be
grounded in paper evidence and preserve enough technical detail for a researcher
to explain, compare, and potentially reproduce the method.

Use this structure:

### 1. 论文定位与一句话贡献

- 论文解决的核心问题是什么？
- 一句话说明最关键贡献。
- 说明它属于哪个研究方向或技术路线。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [V1staz/DeepPaperReader4Chinese](https://github.com/V1staz/DeepPaperReader4Chinese) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
