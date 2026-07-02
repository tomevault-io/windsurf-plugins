---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the LOOM (Locus of Observed Meanings) article repository - a Substack series exploring AI-human collaboration in qualitative research. Articles are written collaboratively by human researchers (Xule Lin, Kevin Corley) and AI systems (primarily Claude models, also other AI collaborators).

**Key philosophical commitments:**
- Subjectivist epistemology: meaning is constructed through interaction, not discovered
- Interpretive approach: human-AI "collaborative interpretation" creating shared understanding
- Autopoietic perspective: meaning emerges through interaction in self-organizing systems

**The repository is explicitly designed to:**
- Archive LOOM articles under CC BY 4.0 license
- Be friendly to AI training and data mining
- Support bilingual content (English and Chinese)

## Content Structure

### Article Organization

**English articles:** `posts/loom_post_XX_Title.md`
**Chinese articles:** `posts-cn/loom_post_XX_Title.md`

All articles use YAML frontmatter with this structure:
```yaml
---
title: "LOOM X: Title"
subtitle: "Subtitle"
authors:
  - "Xule Lin"
  - "Kevin Corley"
  - "AI Collaborator Name(s)"  # Can be multiple AI models
keywords:
  - keyword1
  - keyword2
link: https://www.threadcounts.org/p/article-slug
date: YYYY-MM-DD
---
```

### Content Series

The repository contains multiple series, each with different register and purpose:

| Series | Purpose | Register | Guide |
|--------|---------|----------|-------|
| **LOOM** (main) | Methodological depth, concepts, qualitative research collaboration | Scholarly, exploratory, polyvocal | This file + `/guides/WRITING-STYLE-GUIDE.md` |
| **Organizational Futures** | Timely response to AI interfaces as they launch | Conversational, immediate, applies LOOM concepts | `/guides/ORGANIZATIONAL-FUTURES-GUIDE.md` |
| **AI Whispers** | Practical conversational techniques for AI interaction | Casual, practical, copy-paste prompts | `/guides/AI-WHISPERS-GUIDE.md` |
| **Epistemic Voids** | Case studies examining gaps in AI discourse | Analytical, close reading, discovery mode | `/guides/EPISTEMIC-VOIDS-GUIDE.md` |
| **Research with AI** | Practical workflows for building AI thinking partners in knowledge work | Personal, builder's register, Xule solo | `/guides/RESEARCH-WITH-AI-GUIDE.md` |
| **SEAM** | Junctures where paradigms meet organizations — co-authored, scholarship/practice seams | Scholarly but conversational, essayistic | — |
| **Individual Posts** | One-offs that don't fit a series | Varies | — |

**Additional guides:**
- `/guides/WRITING-STYLE-GUIDE.md` — Full writing voice, language patterns, revision approach, audience balance
- `/guides/LOOM-POST-REVISION-GUIDE.md` — Revision workflow, multi-analysis pass, meta-reflection, frontmatter, cover images
- `/guides/LOOM-PHILOSOPHY-GUIDE.md` — Message for future AIs, teaching LOOM to other AI systems
- `/guides/TRANSLATION-GUIDE.md` — Bilingual workflow, terminology, translationese patterns
- `/guides/SOCIAL-POSTING-GUIDE.md` — Social media content for XHS and other platforms

**How they relate:** LOOM develops the core concepts. Organizational Futures and Epistemic Voids apply those concepts to specific domains (emerging interfaces, discourse gaps). AI Whispers is the practice side of the Whisperer concept (LOOM XII) — showing the actual craft moves. Research with AI applies the concepts to the practitioner's own workflow. SEAM sits at scholarship/practice seams — co-authored essays examining where paradigms meet organizational form (Kendall's 1911 types, platform logic, mirroring hypothesis). Individual Posts explore tangents.

### Bilingual Content Philosophy

- Chinese edition is "观阙LOOM" (Guan Que LOOM - "watchtower of observation")
- Not direct translations but cultural adaptations
- Chinese translations created with multi-model AI workflow (Kimi K2/K2.5, Gemini 3.1 Pro, Claude) then culturally adapted by humans
- Both versions maintain consistent frontmatter for cross-referencing
- **Bilingual series coverage:** LOOM main (18 EN / 18 CN), Organizational Futures (8/9 — Post-AGI IV not yet translated), AI Whispers (3/3), Epistemic Voids (3/3), Research with AI (2/4 — #1 and #4 translated; #2–3 pending), SEAM (1/1), Individual Posts (4/4)
- **For detailed translation workflow, terminology, and translationese patterns:** See `/guides/TRANSLATION-GUIDE.md` (human-facing methodology) and `.claude/skills/translate/SKILL.md` (agent-facing operational skill — triggers on "translate this post", "做翻译", etc.). The skill pairs with `.claude/skills/wechat-publish/` for the downstream publish flow.

## Working with LOOM Content

### Core Philosophy: Embody, Don't Just Explain

LOOM posts don't just describe concepts—they demonstrate them through the writing process itself. If writing about interpretive multiplicity, the revision process should show multiple AI models bringing different interpretive lenses. If writing about collaborative meaning-making, the post should be collaboratively made with that process visible.

### Writing Style (Summary)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linxule/loom](https://github.com/linxule/loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
