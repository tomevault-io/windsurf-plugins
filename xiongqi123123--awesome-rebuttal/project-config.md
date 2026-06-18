---
trigger: always_on
description: Global-installable, project-level academic rebuttal strategy skill for AI/ML/CV/NLP/Robotics papers. Use when authors need a workspace-local .awesome-rebuttal state folder, paper/code/review/venue-rule intake, JSON memory, snapshots, LaTeX/template handling for one-page rebuttals, reviewer stance analysis, strategy planning, experiment triage, safe author response drafting, or AC summaries under confirmed venue rules.
---


# Awesome Rebuttal

Use this skill as a **global-installable, project-level rebuttal workspace assistant**. The installed skill provides reusable procedures and assets; each actual paper/rebuttal workspace gets its own `.awesome-rebuttal/` state folder for memory, snapshots, template state, and logs. Start by understanding the workspace, then collect evidence, persist memory, analyze strategy, and only then draft response text.

## Operating contract

1. Inspect the current workspace before content analysis.
2. Create or use a project-local `.awesome-rebuttal/` state folder; never store runtime memory in the installed skill folder.
3. If the workspace is empty, organize or recommend `Code/`, `Paper/`, `Reference/`, and `Temp/`.
4. If the workspace already contains files, infer the author's organization and adapt non-destructively.
5. Ask how progress should be preserved: `manual_git`, `auto_git`, or `markdown_snapshot_only`.
6. Run the intake gate before analysis. Missing required inputs block drafting.
7. Treat venue rules as user-provided or AI-searched + user-confirmed; never rely on stale built-in venue rules.
8. Keep every factual claim grounded in `paper`, `code`, `review`, `venue_rules`, `user`, or explicit `inference`.
9. Never invent experiments, numbers, citations, reviewer positions, or venue permissions.


## Language policy

- Interaction language: follow the user's language by default for questions, analysis reports, progress updates, and explanations.
- Submission language: final rebuttal artifacts must be written in English unless the user explicitly requests another submission language and the venue permits it.
- Memory: record this choice in `project_memory.language_policy` and mirror any venue-specific exception in `venue_rules.language_policy`.
- Drafting rule: `11_response_writer.md`, `12_template_designer.md`, and `13_ac_summary_writer.md` may discuss plans in the user's language, but author-response text, reviewer replies, AC summaries, OpenReview comments, and PDF rebuttal prose default to English.
- Terminology: preserve exact technical terms, metric names, method names, dataset names, and reviewer wording from the paper/reviews; translate only surrounding explanatory prose when needed.
- If the user provides Chinese strategy notes, convert them into professional English rebuttal prose rather than literal translation.

## Shared questionnaire protocol

Whenever the skill hits missing, ambiguous, or confirmation-dependent information, first summarize what the user already provided and what the workspace evidence shows. Then ask a focused questionnaire instead of guessing.

Read `references/core/user_questionnaire_protocol.md` for the reusable questionnaire pattern. Use it especially in workspace bootstrap and intake, and reuse it later for venue-rule confirmation, experiment feasibility, versioning mode, or any strategy decision that materially changes the output.

Prefer structured choices when possible:

- single-choice for mutually exclusive paths
- multi-select for available inputs or constraints
- short text for pasted rules, reviews, paths, or URLs
- confirmation for inferred workspace maps or AI-found venue rules

Ask only for the smallest missing decision set needed for the next safe step.


## Canonical response modes

Use these exact `response_mode` values across all memory files and capability handoffs:

- `openreview_per_reviewer` — one reply/comment per reviewer thread.
- `unified_limited` — one limited unified response where concerns are merged.
- `pdf_one_page` — one-page PDF/LaTeX rebuttal.
- `global_comment` — one global platform comment/text box.
- `hybrid` — global summary plus per-reviewer replies.
- `openreview_markdown_latex` — OpenReview-style Markdown comment with lightweight LaTeX math.
- `unknown` — not confirmed yet.

Do not introduce aliases such as `per_reviewer`, `global`, `global_text`, `one_page_pdf`, or `markdown_latex_hybrid` in new memory. If user wording uses those terms, normalize to the canonical value and record the original wording in notes if useful.

## Venue rules schema contract

Use `references/memory-schemas/venue_rules.schema.json` as the global rule-memory contract. Venue rules are runtime evidence, not built-in knowledge.

Every `venue_rules.json` should separate:

- `status` and `source`: missing/user-provided/AI-found-pending-confirmation plus URL/path/retrieval notes.
- `response`: canonical `mode`, platform, limits, per-reviewer/global/interactive/AC-summary permissions.
- `formatting`: official template, PDF/LaTeX/Markdown support, figures/tables/appendix, and page-layout constraints.
- `content_permissions`: new experiments/results, links, supplements, code links, references, revision commitments.
- `anonymity`: anonymous requirement, self-citation, acknowledgements, and identity-risk notes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiongqi123123/awesome-rebuttal](https://github.com/xiongqi123123/awesome-rebuttal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
