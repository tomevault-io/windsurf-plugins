---
trigger: always_on
description: >
---


# LaTeX Skill

This skill governs the production of LaTeX output for a researcher in computational and
applied mathematics whose work spans numerical optimisation, deep learning theory, electrical
impedance tomography (EIT), regularisation theory, data-driven inversion, numerical analysis,
PINNs, and related areas. All output must satisfy the mathematical standards of journals such
as SIAM Journal on Scientific Computing, Inverse Problems, Mathematics of Computation, and
Journal of Computational Physics.

---

## Step 1. Classify the Request

Before writing anything, determine which of the **three** output modes applies.
Check for BEAMER MODE first, since it has the strongest surface signals.

### Beamer Mode  ← CHECK THIS FIRST

Produce a complete Beamer presentation (`.tex` file with `\documentclass{beamer}`) when
the request contains any of:

- The words "slides", "presentation", "beamer", "slide deck", "talk", "seminar",
  "conference talk", or "present my work on".
- A request to prepare a talk from a manuscript, paper draft, or set of results.
- A request for a "title slide", "table of contents slide", or named academic sections
  such as "introduction slide", "convergence results slide", "numerical experiments slide".

Proceed to **Step 2B** for the beamer preamble and **Step 3B** for beamer content rules.
Do NOT use the article preamble (Step 2) or article document structure (Step 4) for
Beamer Mode output.

**Content sourcing for Beamer Mode:**
- If the user supplies a complete manuscript or paper: extract content from it faithfully
  to populate each slide section.
- If the user supplies only a title, abstract, or partial notes: generate a fully
  structured template with `\todo{}` placeholders in every slide that lacks content.
- Never fabricate theorems, lemmas, or numerical results that were not provided.

### Document Mode

Produce a complete, standalone `.tex` file (preamble through `\end{document}`) using
`\documentclass{article}` when the request involves:

- A theorem, lemma, proposition, corollary, definition, or remark, with or without proof.
- A multi-step derivation or mathematical argument.
- A convergence analysis or complexity result.
- An algorithm presented alongside its theoretical justification.
- An introduction, related-work survey, or structured academic section.
- A self-contained report, technical note, or preprint draft.

Save the output as a `.tex` file and present it to the user for download.

### Snippet Mode

Produce raw LaTeX body content only (no `\documentclass`, no preamble, no
`\begin{document}`) when the request involves:

- A single equation or aligned system of equations.
- An algorithm or pseudocode block in isolation.
- A numerical results table.
- A TikZ figure or pgfplots graph.
- Any fragment intended to be inserted into the user's own template.

Deliver snippet output as a labelled code block in the chat, not as a file, unless the
user explicitly requests a file.

**When in doubt**, ask: "Should I produce a complete standalone document, a snippet, or a
Beamer presentation?"

---

## Step 2. Apply the Standard Preamble (Document Mode Only)

Use the preamble below verbatim. Do not improvise or abbreviate it.

```latex
\documentclass[11pt,a4paper]{article}

% ---------------------------------------------------------------
% Core mathematics packages
% ---------------------------------------------------------------
\usepackage{amsmath, amssymb, amsthm, mathtools}
\usepackage{bm}            % bold math symbols
\usepackage{dsfont}        % indicator function \mathds{1}

% ---------------------------------------------------------------
% Algorithms
% ---------------------------------------------------------------
\usepackage[ruled,vlined,linesnumbered]{algorithm2e}

% ---------------------------------------------------------------
% Graphics, figures, and plots
% ---------------------------------------------------------------
\usepackage{graphicx}
\usepackage{tikz}
\usepackage{pgfplots}
\pgfplotsset{compat=1.18}
\usepackage{subfigure}     % side-by-side figures
\usepackage{adjustbox}     % fallback rescaling for wide TikZ diagrams

% ---------------------------------------------------------------
% Tables
% ---------------------------------------------------------------
\usepackage{booktabs}
\usepackage{tabularx}
\usepackage{siunitx}       % decimal-aligned columns, SI units

% ---------------------------------------------------------------
% Cross-referencing and citations
% ---------------------------------------------------------------
\usepackage[numbers,sort&compress]{natbib}
\usepackage[colorlinks=true,linkcolor=blue,citecolor=blue,urlcolor=blue]{hyperref}
\usepackage[capitalise,noabbrev]{cleveref}

% ---------------------------------------------------------------
% Typography
% ---------------------------------------------------------------
\usepackage{microtype}
\usepackage{parskip}
\setlength{\parindent}{0pt}
\setlength{\parskip}{6pt}

% ---------------------------------------------------------------
% Page layout
% ---------------------------------------------------------------
\usepackage[margin=2.5cm]{geometry}

% ---------------------------------------------------------------
% Miscellaneous
% ---------------------------------------------------------------
\usepackage{xcolor}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hameefy/claude-latex-skill](https://github.com/hameefy/claude-latex-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
