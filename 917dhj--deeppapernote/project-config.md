---
trigger: always_on
description: Generate a high-quality deep-reading note for a single paper and write it into an Obsidian-style vault. Use when the user gives a paper title, DOI, URL, arXiv ID, Zotero item, or local PDF and wants a polished Markdown note with strong structure, evidence-based analysis, and figure placeholders.
---


# DeepPaperNote

Use this skill when the user wants one outcome:
- read one paper carefully
- generate a high-quality Markdown note
- save the note into an Obsidian-style vault when configured, or into the current workspace when no vault is configured

Chinese trigger examples:
- `给这篇论文生成深度笔记`
- `写一篇高质量论文精读笔记`
- `把这篇文章整理成 obsidian 笔记`
- `读这篇论文并生成 md 笔记`

This skill is intentionally narrow:
- it handles one paper at a time
- it does not update daily reading lists
- it does not treat a shallow abstract rewrite as a successful output
- it does not split the public entrypoint into separate setup, troubleshooting, or start commands

## Core Standard

The finished note must be more than a summary. It should reconstruct the paper's argument:
- what problem it solves
- how the task is defined
- what data or materials it uses
- how the method or analysis actually works
- what results matter most
- what the paper does not prove
- why the paper is worth keeping

Default writer persona:
- a top-tier researcher or algorithm engineer
- writing a replication-oriented lab note
- not writing a popular-science explanation
- assuming the reader can follow Python, PyTorch, training loops, and evaluation logic

The note must adapt to the paper type. Use the same base structure, but shift emphasis for AI methods, benchmarks, clinical studies, and humanities or social-science papers.

## Workflow

Follow this order:
1. resolve the paper identity
2. collect metadata
3. acquire the best available PDF
4. extract canonical raw source text: `*_raw_sections.jsonl`, `*_source_manifest.json`, and optional derived `*_full_text.md`
5. extract structural indexes and PDF assets
6. plan figure placement
7. build the full figure/table decision table
8. build the manifest synthesis bundle
9. have the model read the bundle plus raw sections and plan the note
10. run grounding lint on the note plan before drafting from it
11. have the model write the note
12. lint the final note — if the lint output contains `passes_style_gate: false`, apply the Style Gate Enforcement rule before advancing to step 13, 14, or 15
13. perform `final_quality_review` after lint passes
14. perform `final_readability_review` after the quality review passes
15. write into Obsidian

This is the required workflow for a normal single-paper note request, not a loose suggestion.
Unless this skill explicitly marks a stage as optional, required stages must not be silently skipped, reordered into a shortcut, or treated as complete just because a partial artifact already exists.

Global no-short-circuit rule:
- do not stop after only the early stages and present the workflow as finished
- do not treat slowness, inconvenience, or temporary uncertainty as permission to bypass a required stage
- do not replace the declared workflow with an improvised shortcut
- if a required stage fails, only do one of three things:
  - retry that stage
  - enter a fallback that is explicitly allowed by this skill
  - stop and report which stage is blocked and which downstream required stages remain incomplete
- do not describe the whole task as complete while required downstream stages are still pending

Completion-language rule:
- say `笔记已完成` only when the required workflow is actually complete
- say `已生成草稿` when drafting is done but lint, final readability review, or save is still pending
- say `已通过校验` only when lint has actually been run and passed
- say `已保存到 Obsidian` only when the write step has actually succeeded
- do not treat `lint 已通过` as equivalent to `整篇笔记已经润色完成`
- if final readability review is still pending, explicitly say the draft passed script lint but has not finished final language review
- if the workflow stopped early, name the current stage and the still-missing required stages instead of using completion language
- lint is a floor, not the writing objective

## Core Execution Contract

`SKILL.md` plus the generated `synthesis_bundle.json` must be enough to complete a normal note-generation run.
Files under `references/` are optional stage-specific deep dives, not a default reading checklist.

Non-negotiable rules:
- evidence-first: draft from the synthesis bundle, `source_manifest`, raw sections, coverage metadata, explicit `note_plan`, and inspected paper evidence; never finish from title/abstract/headings alone
- raw-source authority: for ordinary PDFs, `*_raw_sections.jsonl` and `*_source_manifest.json` are the canonical reading material; old top-N evidence buckets, truncated `section_texts`, and `candidate_chunks` are not model-facing writing inputs
- fail-closed: if a usable PDF or sufficient evidence cannot be obtained after supported acquisition paths, stop and ask for better source material rather than producing a finished degraded note
- model-first: scripts structure evidence, but the model must decide emphasis, contribution, mechanism, limitations, and final Chinese prose

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [917Dhj/DeepPaperNote](https://github.com/917Dhj/DeepPaperNote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
