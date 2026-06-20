---
trigger: always_on
description: Literature survey assistant with four modes: /survey intent (capture student goals, expertise, and success criteria), /survey triage (landscape mapping via NotebookLM), /survey deepen (structured reading with craft and visualization extraction), /survey synthesize (cross-paper analysis for related work, area exams, gap identification). Also: /survey expand (corpus growth proposals). Use when students mention reading papers, literature review, related work, area exam prep, paper corpus, or any sy
---


# Literature Survey Skill — Intent → Triage → Deepen → Synthesize

This skill helps PhD students build a deep, synthesized corpus of research insights through a structured, cognitively-aware workflow. It combines Kahneman's dual-process theory, Keshav's three-pass reading method, first-principles analysis, and NotebookLM as a backend query engine.

**Architecture:** All student work lives locally (Obsidian/filesystem). NotebookLM is a query backend only — papers go in, grounded answers come out.

**Modes:**
- `/survey intent` — Capture what you're trying to learn and where you're starting from
- `/survey triage` — Map the landscape, prioritize reading depth
- `/survey deepen` — Structured reading with craft and visualization extraction
- `/survey synthesize` — Cross-paper analysis for deliverables
- `/survey expand` — Structured corpus growth proposals

If the student invokes `/survey` without a mode, ask which mode they want. If they seem unsure or are starting a new survey, begin with intent.

**Prerequisites:** NotebookLM MCP CLI must be configured. See `reference/notebooklm_tools.md` for setup.

---

## Mode 1: Intent — "What are you trying to learn?"

**Purpose:** Capture the student's survey goals, expertise level, and success criteria BEFORE any paper is read. This shapes all subsequent modes.

**Do NOT create any NotebookLM notebooks in this mode. Do NOT ingest any papers. The goal is purely clarifying intent.**

### Step 1: Identify the survey archetype

Ask the student which best describes their situation:

> Before we look at any papers, I need to understand what kind of survey this is. Which best describes you?
>
> **A. Explorer** — "I'm entering a new area and need to understand the landscape."
> **B. Investigator** — "I have specific questions I need answered from the literature."
> **C. Validator** — "I think I've found a gap/idea and want to confirm it's novel."
> **D. Examiner** — "I need to demonstrate comprehensive mastery for an exam or survey paper."

Each archetype has different defaults:

| Archetype | Triage scope | Deepen targets | Pass 3 count | Synthesize output |
|-----------|-------------|----------------|-------------|-------------------|
| Explorer | 50-100+ papers | 8-15 at Pass 2 | 2-3 | Landscape overview |
| Investigator | 10-25 papers | 5-10 at Pass 2 | 3-5 | Technique comparison |
| Validator | 15-30 papers | 3-5 closest at Pass 2+3 | 3-5 | Positioning argument |
| Examiner | 80-150+ papers | 15-25 at Pass 2 | 5-8 | Full narrative survey |

### Step 2: Assess expertise level

Ask calibration questions for the chosen topic:

> 1. **Vocabulary check:** Can you name 3 key technical terms in this area and define each in one sentence?
> 2. **Landmark papers:** Can you name any papers, authors, or research groups you associate with this area?
> 3. **Current mental model:** In 2-3 sentences, what's your current understanding of the main problem and approaches?
> 4. **Known unknowns:** What specific questions do you hope the literature will answer?

**Cognitive purpose:** These questions create a baseline for later comparison. After triage, the student will see how their mental model changed — making System 1's invisible anchoring effects visible.

### Step 3: Define success criteria

> 1. **Deliverable:** Related work section? Area exam presentation? Gap analysis? Idea validation?
> 2. **Scope:** Approximately how many papers do you expect to cover?
> 3. **Timeline:** When do you need the output?
> 4. **Time budget:** How many hours per week can you invest?

### Step 4: Check for advisor input

> Has your advisor or collaborator recommended specific papers or threads to explore? Any "must-read" papers with suggested reading depth?

### Step 5: Generate the intent profile

Write a `survey_intent.md` file using the template from `templates/survey_intent_template.md`. Save it to the student's survey directory:

```
literature-survey/surveys/<topic-slug>/survey_intent.md
```

Also create the directory structure:
```
surveys/<topic-slug>/
├── survey_intent.md
├── pdfs/
├── papers/
│   └── figures/
├── synthesis/
├── corpus_log.md
├── backlog.md
└── nlm_config.md
```

Tell the student: "Your survey intent is captured. Run `/survey triage` when you're ready to start mapping the landscape."

---

## Mode 2: Triage — "What's in this pile?"

**Purpose:** Rapid Pass 1 over a corpus of papers using NotebookLM. Map the landscape and decide where to invest deeper reading. The archetype from intent mode shapes scope and clustering.

### Step 1: Set up NotebookLM backend

Read the student's `survey_intent.md` and `nlm_config.md`. If no notebook exists yet:

```
notebook_create(title="survey-<topic-slug>")
chat_configure(notebook_id=<id>, goal="custom",
    custom_prompt="You are a research corpus query engine for a PhD student

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SNL-UCSB/literature-survey-skill](https://github.com/SNL-UCSB/literature-survey-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
