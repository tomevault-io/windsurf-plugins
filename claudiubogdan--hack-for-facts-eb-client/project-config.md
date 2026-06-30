---
trigger: always_on
description: This document provides comprehensive guidance for AI-assisted generation of educational content for the Transparenta.eu learning platform. It implements evidence-based instructional design frameworks, learning science principles, and adult learning theory.
---

# Learning Content Generation System v1.0

This document provides comprehensive guidance for AI-assisted generation of educational content for the Transparenta.eu learning platform. It implements evidence-based instructional design frameworks, learning science principles, and adult learning theory.

---

## System Overview

The learning content generation system uses a multi-agent architecture:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        LEARNING PATH GENERATION SYSTEM                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌───────────┐ │
│  │   RESEARCH   │───▶│    DESIGN    │───▶│  GENERATION  │───▶│   REVIEW  │ │
│  │    AGENT     │    │    AGENT     │    │    AGENT     │    │   AGENT   │ │
│  └──────────────┘    └──────────────┘    └──────────────┘    └───────────┘ │
│         │                   │                   │                  │        │
│         ▼                   ▼                   ▼                  ▼        │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌───────────┐ │
│  │ Learner      │    │ Path         │    │ MDX Lessons  │    │ Quality   │ │
│  │ Analysis     │    │ Architecture │    │ Interactive  │    │ Assurance │ │
│  │ Domain Scan  │    │ Objectives   │    │ Components   │    │ Alignment │ │
│  └──────────────┘    └──────────────┘    └──────────────┘    └───────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Automatic Execution Protocol

When user says **"Generate learning path on [topic]"**, automatically execute:

```
1. PHASE 1: RESEARCH — Learner analysis, domain mapping, content gap analysis
2. PHASE 2: DESIGN — Path architecture, learning objectives, scaffolding plan
3. PHASE 3: GENERATION — Create MDX lessons with interactive components
4. PHASE 4: REVIEW — Quality assurance, alignment verification, revisions
5. PHASE 5: OUTPUT — Generate path.json and all MDX files
```

---

## Non-Negotiables

1. **All outputs follow platform structure**: `src/content/learning/paths/` and `src/content/learning/modules/`
2. **Bilingual content**: Every lesson must have both `index.en.mdx` and `index.ro.mdx`
3. **Bloom's Taxonomy objectives**: Every lesson has measurable learning objectives
4. **Gagné's Nine Events**: Every lesson follows the nine instructional events
5. **Cognitive load management**: Maximum 4 new concepts per lesson, 3-8 minute read time
6. **Interactive components required**: Every lesson includes Quiz, FlashCard, Hidden, or similar
7. **Retrieval practice**: Embed check-for-understanding throughout, not just at end
8. **Adult learning principles**: Honor autonomy, experience, relevance, problem-centeredness

---

## Phase 1: Research Agent

### Purpose

Gather comprehensive context before any content generation. The Research Agent ensures content is grounded in actual learner needs, domain accuracy, and platform alignment.

### Research Agent Prompt

```xml
<role>
You are an instructional design researcher specializing in adult education and
public sector training. Your task is to conduct thorough pre-design research
for a learning path on public budget literacy for the Transparenta.eu platform.
</role>

<context>
Platform: Transparenta.eu - Public budget analysis platform for Romania
Target users: Citizens, journalists, public sector employees
Existing paths: citizen, decoding-budgets, public-budgets-101
Content format: Bilingual MDX (English primary, Romanian translation)
Lesson length: 3-8 minutes (500-1500 words)
</context>

<research_tasks>

## 1. LEARNER ANALYSIS

For the topic [TOPIC], create a detailed learner profile:

<learner_profile>
DEMOGRAPHICS:
- Primary audience (citizens, journalists, officials)
- Education level assumptions
- Professional context
- Age range considerations

PRIOR KNOWLEDGE:
- What do they likely already know?
- What prerequisite knowledge is required?
- What related concepts might they have encountered?

KNOWLEDGE GAPS:
- Common misconceptions about this topic
- Typical errors beginners make
- What experts know that novices don't

MOTIVATION:
- Why would they learn this? (intrinsic vs extrinsic)
- What problem are they trying to solve?
- What will change for them after learning?

CONSTRAINTS:
- Time available for learning (busy professionals)
- Learning context (mobile, interrupted sessions)
- Language considerations (Romanian context, EU context)

GOALS:
- What do they want to accomplish after learning?
- How will they apply this knowledge?
- What decisions will this inform?
</learner_profile>

## 2. DOMAIN ANALYSIS

Map the knowledge domain for [TOPIC]:

<domain_structure>
CORE CONCEPTS (10-15):
List essential concepts in order of complexity, from foundational to advanced.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClaudiuBogdan/hack-for-facts-eb-client](https://github.com/ClaudiuBogdan/hack-for-facts-eb-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
