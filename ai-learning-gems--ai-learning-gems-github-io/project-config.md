---
trigger: always_on
description: WCAG-compliant semantic color-coding for key concepts in textbook chapters — applies during editing pass
---


# Semantic Concept Coloring

Rules for applying consistent semantic color-coding to key concepts throughout textbook chapters. This technique uses author-provided color as a **visual signal** (Mayer's Signaling Principle) to help readers mentally separate and track concept categories.

**Research basis:** Color-coding is a form of visual signaling with meta-analytic support (Alpizar et al. 2020, g = 0.22; Richter et al. 2016, d = 0.22). Liu et al. (2021) found large effect sizes (eta-squared = 0.23) with lower cognitive load (measured by pupil diameter and EEG). This is fundamentally different from student-directed highlighting (Dunlosky LOW utility): author-provided semantic color-coding is expert-curated, consistent, and functions as a perceptual signal, not a learning activity. It leverages pre-attentive processing (Treisman & Gelade 1980), the Von Restorff isolation effect, and dual coding (Paivio 1986).

---

## When to Apply

The coloring pass is part of the **editing workflow** (`edit-textbook-chapter.md`). It runs AFTER prose-quality editing is complete — it should be the last content-level pass before final verification. The writing workflow (`write-textbook-chapter.md`) may also apply coloring during initial writing, especially in the Chapter Overview paragraph where the technique has the highest impact.

---

## The Color Palette (WCAG AA Compliant)

All colors pass WCAG AA (4.5:1 minimum) for normal-size text on white backgrounds. They are Tailwind CSS 600-700 shades, chosen for perceptual distinguishability and colorblind safety.

| # | Name | Hex | WCAG Ratio | Prose Syntax | LaTeX Syntax |
|---|---|---|---|---|---|
| 1 | **Indigo** | `#4F46E5` | 6.29:1 | `[**term**]{style="color: #4F46E5;"}` | `$\textcolor{#4F46E5}{symbol}$` |
| 2 | **Emerald** | `#047857` | 5.48:1 | `[**term**]{style="color: #047857;"}` | `$\textcolor{#047857}{symbol}$` |
| 3 | **Rose** | `#E11D48` | 4.70:1 | `[**term**]{style="color: #E11D48;"}` | `$\textcolor{#E11D48}{symbol}$` |
| 4 | **Sky** | `#0369A1` | 5.93:1 | `[**term**]{style="color: #0369A1;"}` | `$\textcolor{#0369A1}{symbol}$` |
| 5 | **Amber** | `#B45309` | 5.02:1 | `[**term**]{style="color: #B45309;"}` | `$\textcolor{#B45309}{symbol}$` |
| 6 | **Purple** | `#7E22CE` | 6.98:1 | `[**term**]{style="color: #7E22CE;"}` | `$\textcolor{#7E22CE}{symbol}$` |
| 7 | **Teal** | `#0F766E` | 5.23:1 | `[**term**]{style="color: #0F766E;"}` | `$\textcolor{#0F766E}{symbol}$` |
| 8 | **Slate** | `#475569` | 7.58:1 | `[**term**]{style="color: #475569;"}` | `$\textcolor{#475569}{symbol}$` |

**Use at most 3-5 colors per chapter.** Pick the colors most relevant to the chapter's concept categories. The full palette of 8 provides flexibility across different chapters, but She et al. (2024) showed single-color cues outperform multi-color, and the Von Restorff effect disappears when everything is colored.

---

## How to Assign Colors to Concepts

### Step 1: Identify 3-5 Major Concept Categories

Read the chapter overview and identify the 3-5 most important conceptual pillars. These should be genuinely distinct categories, not synonyms or subcategories of each other.

**Example (MoE chapter):**

| Color | Category | Example Terms |
|---|---|---|
| Indigo `#4F46E5` | Architecture/structure | MoE layer, expert FFN, dense vs sparse |
| Emerald `#047857` | Routing/selection | router, top-k, gating, token assignment |
| Rose `#E11D48` | Training/optimization | auxiliary loss, load balancing, capacity factor |

**Example (Vision Transformers chapter):**

| Color | Category | Example Terms |
|---|---|---|
| Indigo `#4F46E5` | Architecture | ViT, patch embedding, class token |
| Emerald `#047857` | Attention mechanism | self-attention, multi-head, Q/K/V |
| Amber `#B45309` | Training/scaling | pre-training, fine-tuning, data augmentation |
| Sky `#0369A1` | Mathematical objects | patch vectors, position embeddings, attention weights |

### Step 2: Document the Color Map

At the top of TEXTBOOK-PLAN.md (or in the editing pass notes), create an explicit color map table. This is the single source of truth for the chapter's coloring. Every colored term must trace back to this table.

### Step 3: Never Reuse a Color for a Different Category

If Indigo means "architecture" in Section 1, it must mean "architecture" in Section 6. Inconsistency turns signal into noise and violates the categorization benefit.

---

## Where to Apply Color

### In the Chapter Overview (HIGHEST IMPACT)

The Chapter Overview paragraph is the ideal place for the first appearance of colored terms. This is where the reader builds their mental model of the chapter's structure, and color visually separates the pillars.

**Example:**

```markdown
To understand MoE, you need a mental model with three layers: (1) the
[**architecture**]{style="color: #4F46E5;"}, i.e. what an MoE layer looks like;
(2) the [**routing mechanism**]{style="color: #047857;"}, i.e. how the model decides
which expert handles which token; and (3) the
[**training machinery**]{style="color: #E11D48;"}, i.e. the auxiliary losses and
balancing tricks that make MoE training stable.
```

### On First Mention of a Key Term in Each Section


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AI-Learning-Gems/AI-Learning-Gems.github.io](https://github.com/AI-Learning-Gems/AI-Learning-Gems.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
