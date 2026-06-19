---
trigger: always_on
description: Research paper writing assistant that enforces Arpit Gupta's editorial principles, voice profile, and writing workflow. MANDATORY TRIGGERS: Use this skill whenever the user mentions writing a paper, drafting a section, revising a section, editing a paper, reviewing a draft, rewriting an introduction, writing an evaluation, polishing prose, compressing text, or any task involving .tex files, Overleaf, conference submissions, or paper deadlines. Also trigger when the user mentions any paper by nam
---


# Paper Writing Skill

## How This Skill Works

This skill encodes the writing methodology of the [Systems and Networking Lab (SNL)](https://github.com/SNL-UCSB) at UC Santa Barbara, derived from forensic analysis of 6 papers (8 submissions), 7,600+ Overleaf edits, 100+ tex file versions, and 5 peer review processes. See [*The Paper Behind the Paper*](https://sites.cs.ucsb.edu/~arpitgupta/blog/the-paper-behind-the-paper.html) for the full analysis. It works out of the box — the default rules are calibrated and battle-tested.

### Three Layers

1. **The pipeline (fixed)**: A five-stage writing workflow. Does not change between users or papers.

2. **The voice and editorial rules (defaults provided, customizable)**: Sentence-level style, structural rules, compression patterns, section checklists. These ship with the SNL lab's rules as defaults. Students may customize by editing files in `author_profile/` — see the README for what to change.

3. **The project context (per paper)**: Identity sentence, venue, contribution claims, locked decisions. Lives in a `project_context.md` in the paper's working directory.

### How This Skill Connects to the Research Pipeline

This skill does not operate in isolation. It is part of a three-skill family, and the artifacts from the other two skills are direct inputs to the writing process:

**From the [literature-survey-skill](https://github.com/SNL-UCSB/literature-survey-skill):**
- **Gap analysis** → feeds Brainstorming Phase 1 (Problem Discovery). The gaps the survey identified — missing quadrants, shared assumptions that break, unexplored combinations — are the structural limitations that motivate your paper.
- **Writing craft extractions** (Pass 3+) → feed the Architecture stage and section drafting. The introduction anatomy, evaluation architecture, and design craft you extracted from the best papers in your area are the models for your own paper's structure.
- **Competitive positioning** → feeds Brainstorming Phase 4. The invariant matrix and dependency graph from synthesis show exactly where your paper sits relative to existing work.

**From the [data-visualization-skill](https://github.com/SNL-UCSB/data-visualization-skill):**
- **Exploration** (`exploration_log.md`) → feeds Brainstorming Phase 3 (Evaluation Design). The exploration forced you to look at your data from multiple angles before forming hypotheses. The surprises you found — distributions you didn't expect, subgroups that behaved differently — shape what claims are defensible and where the real contribution lives.
- **Brainstorm** (`braindump.md`) → feeds the figure/table plan. Each braindump articulates what question a figure answers, what you expected to see, and what would surprise you. These are the hypotheses your evaluation must validate.
- **Plan + Execute** (`plot_context.md`) → feeds the Architecture stage's figure/table plan. Each plot_context records intent, variable mappings, plot type rationale, and design decisions — ready-made entries for the paper's figure plan.
- **Analyze** (WALTER narrations) → feeds Evaluation Move 4 (Takeaway Synthesis). The WALTER Result — "what is the takeaway? does it connect back to the hypothesis?" — is a first draft of the Takeaway paragraph for that experiment cluster.

The three skills create a closed loop: the literature survey reveals the gap and teaches you how accepted papers communicate; data visualization forces you to understand what your evidence actually shows and what hypotheses it validates; paper writing turns both into a publishable argument. **If the student has artifacts from the other skills, Claude MUST load them.**

### When This Skill Triggers, Claude MUST:

1. Read this SKILL.md (already loaded)
2. Read ALL files in `author_profile/` — these are the source of truth for editorial rules
3. Ask which paper the user is working on
4. Look for a `project_context.md` in the paper's working directory
5. If found, read it and treat it as binding constraints
6. If not found, run the **Structured Brainstorming** workflow below to create one
7. Check for artifacts from sibling skills — survey paper notes with craft extractions, `exploration_log.md`, `braindump.md`, `plot_context.md`, WALTER narrations. If found, load them as reference material for the relevant pipeline stages

---

## Structured Brainstorming — The Skill's Centerpiece

**The biggest obstacle for students isn't writing — it's that their ideas live as unstructured intuitions.** They know something is interesting but can't articulate what or why. The brainstorming process transforms scattered thinking into a precise project context that drives every section of the paper.

### How It Works

Claude MUST read `brainstorming_guide.md` and walk the student through its 6 phases interactively. The phases are:

| Phase | Focus | Key outcome |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SNL-UCSB/paper-writing-skill](https://github.com/SNL-UCSB/paper-writing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
