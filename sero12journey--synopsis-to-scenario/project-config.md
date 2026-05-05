---
trigger: always_on
description: A multi-agent workflow engine that converts various source materials (short-form, web drama, novel, webtoon, real stories) into Korean feature-length film screenplays. Final output: completed Korean screenplay (.docx / .pdf).
---

# Synopsis to Screenplay Workflow Engine

## Project Overview

A multi-agent workflow engine that converts various source materials (short-form, web drama, novel, webtoon, real stories) into Korean feature-length film screenplays. Final output: completed Korean screenplay (.docx / .pdf).

## Architecture

Four agents, strict separation of concerns:

| Agent | Role | Steps |
|-------|------|-------|
| **Orchestrator** | Workflow control, data passing between steps, checkpoint management, human feedback routing. Never writes creatively. | All |
| **Analyst** | Story architect — synopsis analysis, act structure, beat sheet design | 0, 1, 2 |
| **Writer** | Screenwriter — image system, character design, treatment, scene list, first draft | 3, 4, 5, 6, 7 |
| **Critic** | Script doctor — structural diagnosis, subtext detection, image system consistency, visual revision | Checkpoints A/B/C pre-review, Step 8 |

**Design principle**: Writer never evaluates its own output (self-confirmation bias). Critic reviews first, then Human decides.

## Workflow Pipeline

```
STEP 0  [Analyst]  Synopsis pre-analysis (Critique)
STEP 1  [Analyst]  Act structure & duration design
STEP 2  [Analyst]  Beat sheet (15 beats, Snyder-based)
        [Critic → Human]  Checkpoint A — Structure locked
STEP 3  [Writer]   Image system design
STEP 4  [Writer]   Character redesign
        [Critic → Human]  Checkpoint B — Visual strategy & characters locked
STEP 5  [Writer]   Treatment (15-20 pages)
STEP 6  [Writer]   Scene list (70-85 scenes for 90min, derived from treatment)
        [Critic → Human]  Checkpoint C — Pre-draft final review
STEP 7  [Writer]   First draft (55-70 pages)
STEP 8  [Critic → Writer]  Visual revision (Linda Seger process)
```

## Data Flow Rules

- STEP 0 critique feeds into ALL subsequent steps
- Steps are sequential except STEP 3 & 4 (can run in parallel, cross-reference)
- Each checkpoint gates the next phase — no skipping
- Human feedback at checkpoints can route back to any prior step for rework

## Korean Screenplay Conventions

- **Primary metric**: Scene count (NOT page count) — Korean scripts have no standardized font/layout
- **90min film**: 70-85 scenes, 55-70 pages
- **Page-to-minute ratio**: 1.3-1.5 (NOT 1:1 like Hollywood Courier 12pt)
- **Format terms**: 대지문 (master action), 소지문 (sub-action), scene heading, dialogue
- **Aesthetic**: 여백의 미학 (beauty of white space) — leave room for director's interpretation

## Writing Principles

- Tony Tost: "Every sentence is one shot, every period is a cut"
- Trottier: "Write only what can appear on screen"
- McKee: Image systems must be subliminal — if the audience notices the symbol consciously, it's preaching
- Action lines: max 3-4 lines
- Present tense, active voice, strong verbs
- Show don't tell: "He is lonely" → "He eats a convenience store lunch box alone"

## Core Analysis Framework (STEP 0)

Six universal improvement areas to check against any source material:

1. **Psychological journey depth** — inner transformation, not just external conflict
2. **Midpoint genre subversion** — tone/genre flip at the exact middle
3. **Invisible plants** — foreshadowing that only becomes visible in retrospect
4. **Dimensional antagonists** — no monolithic evil, use implication over exposition
5. **No lazy exposition** — replace dialogue dumps with visual discovery
6. **Source format → feature pacing** — add breathing room, earn emotional investment

## Theoretical References

| Theorist | Key Concept | Applied In |
|----------|-------------|------------|
| Syd Field | 3-act structure | STEP 1 |
| Blake Snyder | Beat sheet, visual bookends | STEP 2, 3 |
| Robert McKee | Image system, symbolic ascension | STEP 3, 8 |
| David Trottier | Screen-writable content only | STEP 7, 8 |
| Michael Hauge | Visual detail character intros | STEP 4 |
| Linda Seger | Visual revision process | STEP 8 |
| Tony Tost | Virtual shot list (sentence=shot) | STEP 7 |
| Bong Joon-ho | Architectural space metaphor, 전(轉) | STEP 3, 4, 5 |
| David Mamet | "Write a silent film" test | STEP 8 |

## Reference Documents

프롬프트에서 상세 사례/이론이 필요할 때 참조하는 레퍼런스:

| File | Key Content | Used By |
|------|-------------|---------|
| `영화시나리오연구_영화적묘사_markdown.md` | 영상적 묘사 7가지 기법, Tony Tost virtual shot list, McKee 이미지 시스템 사례, 서브텍스트 번역, 한국 시나리오 특성, 숏폼 적용 | Writer (STEP 3,4,7), Critic (STEP 8) |
| `영화 시나리오 작법 연구 개요.docx` | 구조 이론 (Campbell/McKee/Field/Snyder/Truby), Unity Arc, 복선-회수 기법, 피해야 할 클리셰, 명작 사례 (기생충/EEAAO/헤어질 결심), AI 시나리오 생태계 | Analyst (STEP 0), Writer (STEP 4,5) |

## ProjectConfig Schema

```yaml
title: string              # Working title
source_format: enum        # shortform | webdrama | novel | webtoon | real_story | original
target_format: feature_film
target_duration: 90        # Target runtime in minutes (default 90)
genre: string[]            # Genre tags
tone: string               # Tone description
theme_direction: string    # Thematic direction
reference_docs: file[]     # Screenwriting reference documents
synopsis_doc: file         # Source synopsis
```

## Step Output Schema

Every step produces:

```yaml
step_number: number

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sero12journey/synopsis-to-scenario](https://github.com/sero12journey/synopsis-to-scenario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
