---
trigger: always_on
description: Open-community research-paper and IEEE LaTeX writing support for most IEEE fields, including IEEEtran journals, conferences, letters, magazines, robotics, reinforcement learning, control, intelligent systems, communications, signal processing, power, energy, and related venues. Use when Codex needs to draft, polish, anonymize, audit, or strategically reshape IEEE-style research manuscripts; improve scientific narrative, contribution coherence, reviewer awareness, experimental rigor, domain-speci
---


# IEEE LaTeX Writer Open

Open-community skill for writing, revising, anonymizing, and auditing IEEE-style LaTeX manuscripts across most engineering and computing fields. Treat paper strategy and scientific argument quality as first-class tasks, then apply venue style, domain conventions, LaTeX mechanics, and static checks.

## Core Workflow

1. Identify the target venue, IEEE society/field, manuscript type, page limit, review mode, and submission stage: first submission, double-blind review, revision, camera-ready, or archive cleanup.
2. Verify format-sensitive requirements when they may have changed: venue page, IEEE Template Selector, IEEE Author Center, page limits, anonymity policy, preprint/arXiv policy, supplementary-material rules, and bibliography requirements.
3. Work in IEEEtran-compatible LaTeX. Do not modify margins, fonts, title spacing, bibliography style, or class internals unless the venue explicitly allows it.
4. Identify the paper strategy before line editing: core novelty, likely reviewer concerns, missing evidence, weak ablations, venue fit, and whether secondary ideas dilute the main contribution.
5. Route the manuscript through the relevant venue and domain modules before polishing: robotics/control, RL robotics, computer science/intelligent systems, communications/signal processing, power/energy systems, or general IEEE engineering.
6. Improve the scientific argument first: claim, gap, method, evidence, limitation, contribution, and reproducibility. Then polish style, notation, and LaTeX mechanics.
7. Preserve user source structure. Avoid overwriting macros, comments, figure paths, bibliography databases, or package choices unless they conflict with IEEE requirements.
8. For double-blind review, run an anonymity pass before style polishing. Detect and flag author names, affiliations, acknowledgments, grant numbers, ORCID IDs, repository links, institutional URLs, lab-specific datasets, and distinctive equipment/software descriptions.
9. Rewrite self-citations in third person. Use forms such as "In [1], Smith et al. developed..." or "The method in [1]..." and never "In our previous work [1]..." during double-blind review.
10. For revision tasks, create a response letter with a fixed mapping for every reviewer item: `Reviewer's Comment`, `Response`, and `Changes in the Revised Manuscript`. Keep the tone polite, respectful, specific, and evidence-based.
11. For citation work, verify the formal publication version, DOI, venue, pages/article number, publisher page, and authorized/open access status before inserting or rewriting BibTeX.
12. Run compilation or static audit when files are available. If compilation is unavailable, run or simulate `scripts/audit_ieee_latex.py <project-or-main.tex>` and report residual risks.

## Resource Map

- Read `references/ieee-style-guide.md` for IEEE writing, citation, figure, table, equation, anonymity, BibTeX, and domain-notation guidance.
- Read `references/latex-project-workflow.md` when creating or repairing a LaTeX project, choosing compile commands, preparing an archive, or explaining the static audit spectrum.
- Read `references/revision-and-review.md` when revising a draft, building a response letter, mapping reviewer comments to manuscript changes, or marking changed text.
- Use `assets/ieee-official-templates/` as the bundled IEEEtran starter package when a local template is needed. Prefer the target venue's current template package or IEEE Template Selector for real submissions, then fall back to the bundled official journal sample.
- Run `scripts/audit_ieee_latex.py <project-or-main.tex>` for lightweight static checks. Treat it as a preflight aid, not a replacement for compilation or official IEEE validation tools.
- Run `scripts/clean_ieee_bib.py <references.bib>` when the user wants a deterministic first pass over exported BibTeX before manual DOI, venue, and formal-version verification.

## Paper Strategy Layer

Before drafting or revising, identify the paper's strategic shape:

- State the core novelty in one sentence. Separate it from supporting modules, implementation details, and replaceable engineering components.
- Map the title, abstract, introduction, method, experiments, and conclusion to the same central claim. If any section sells a different paper, revise it.
- Identify likely reviewer attacks: missing baselines, weak ablations, scalability, sim-to-real gap, hardware fragility, unclear novelty, statistical weakness, excessive complexity, or overclaiming.
- Suppress secondary ideas that dilute the main contribution. Move them to ablations, implementation details, appendices, or limitations unless they are essential to the claim.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Listen-Sun/ieee-latex-writer](https://github.com/Listen-Sun/ieee-latex-writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
