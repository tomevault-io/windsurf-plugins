---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Repository purpose

This repository is for authoring and collecting agent skills focused on research workflows, industrial R&D, paper writing, PhD applications, language test preparation, presentation/deck building, and implementation-heavy coding work. Skills should help agents support research ideation, literature grounding, industry problem framing, solution design, pilot evaluation, implementation, reproducibility, paper writing, reviewer response, artifact release, research-oriented applications, IELTS preparation, slide/deck production, and coding support.

Coding-support skills are allowed only when they encode something a capable agent does *not* already do by default; generic craft guidance (style, testing patterns, standard debugging) is explicitly out of scope and was removed from this repo. Generic README, architecture, slide, or meta-discovery skills should not be added unless they directly support research, industrial R&D, paper production, PhD applications, language test preparation, presentation/deck building, or coding-heavy research work.

## Structure

- `skills/` contains individual skills. Each skill should live in its own directory and include a `SKILL.md` file.
- `DESIGN-PHILOSOPHY.md` records the repo's durable skill-design rationale and supporting references. When adding, removing, or materially changing skill-family philosophy, learning/audit protocols, role boundaries, or evidence-backed design claims, update this file if the change affects the rationale.
- Skills are filed at `skills/<category>/<skill>/SKILL.md` — **at most two levels under `skills/`**. This is not cosmetic: the skills CLI's default discovery stops at that depth, so a skill nested deeper is invisible to anyone installing without `--full-depth`. Do not create sub-categories inside a category; group siblings with a name prefix (`theory-*`, `apply-*`, `banking-*`, `flow-*`) instead.
- `skills/init-workspace/` sits at the top level *deliberately*, as the entrypoint skill for starting any new project. Top-level placement is reserved for that role; everything else belongs in a category.
- `skills/research-discovery/` contains early-stage research framing skills, plus `gap-finder` (mining others' papers for one's own research gaps — discovery work, not claim-checking; it also owns the weekly lemma-autopsy habit: weaken one hypothesis, predict the first failed proof line, construct the smallest witness, and only then compare with the authors' discussion). It also holds `cross-domain-analogy-finder`, a high-variance foraging skill that finds a concept's structural twin in a distant field — either to borrow a portable solution (offense) or to check whether an idea already exists under another name (defense) — gated by a four-way classifier (identical / portable / twin-with-a-delta / surface-only) and an import-a-result test; it grounds prior-art claims through `literature-triangulation` and hands a twin's residual mismatch to `gap-finder`. It is for distant-field structural transfer, not near-field paper search or area survey (both `literature-triangulation`) or stress-testing one chosen idea (`research-idea-stress-test`). The directory also holds `gap-motivation-builder`, the Socratic bridge between finding a gap and proposing a solution: it takes one gap (from `gap-finder` or anywhere else) and interrogates the five-link motivation chain (consequence → beneficiary → blocker → timeliness → payoff), refuses "understudied" as motivation (gap-spotting vs problematization), runs the why-hasn't-this-been-done trichotomy (trivial / done under another name / genuine blocker), and converges on a verdict (MOTIVATED / CONDITIONALLY MOTIVATED / WEAK / UNMOTIVATED / NOT A GAP) plus solution desiderata — requirements any future method must satisfy, each traced to a chain link, including at least one anti-desideratum — while never proposing the method itself. Keep the pipeline boundaries: `gap-finder` finds gaps, `gap-motivation-builder` motivates one and derives desiderata, `research-idea-stress-test` attacks the chosen solution direction, `paper-idea-and-scope-brainstormer` scopes the paper, and `abstract-and-intro-writer` turns a finished motivation into prose.
- `skills/research-evidence/` contains literature mapping, Related Work writing, standalone survey/tutorial/perspective paper writing, and citation verification skills. Keep the boundary clear: `related-work-writer` is for Related Work sections inside original research papers, while `survey-paper-writer` is for papers whose main contribution is the synthesis/taxonomy/tutorial/roadmap itself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jurgendn/agent-skills](https://github.com/jurgendn/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
