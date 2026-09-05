---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the myBibleToolbox project - an initiative to create the largest AI-readable commentary on the entire Bible. The goal is to provide extensive context data for AI systems to ground their responses in truth when working with Biblical texts, especially for Bible translators, pastors, and students.

### Theological Foundation: Conservative Protestant Christian

**CRITICAL**: This is a CHRISTIAN project, grounded in conservative Protestant orthodoxy. All theological content follows this framework:

**1. Primary Perspective: Historic Christian Orthodoxy**
- Conservative Protestant Christian is the PRIMARY theological perspective
- Grounded in Scripture as God's inerrant Word
- Affirms historic Christian creeds (Nicene, Apostles', Athanasian)
- Core doctrines: Trinity, deity of Christ, salvation by grace through faith, biblical authority

**2. Christian Denominational Variations**
- Show views of ALL Christian traditions for consideration:
  - Protestant (various denominations)
  - Catholic (Roman Catholic)
  - Orthodox (Eastern Orthodox, Oriental Orthodox)
  - Coptic
  - Other historic Christian traditions
- Present these as VALID CHRISTIAN perspectives worth considering
- Note where they differ on non-essential matters while affirming shared orthodoxy

**3. Non-Orthodox Views (Cults and Other Religions)**
- May note alternate views from:
  - Cults claiming to be Christian (Jehovah's Witnesses, Mormons, etc.)
  - Non-Christian religions (Judaism, Islam, etc.)
- MUST explain WHY these are rejected by Christian orthodoxy
- Label clearly as "not recognized by Christian orthodoxy"
- Purpose: Help translators avoid theological errors, understand why certain interpretations are problematic

**Language to Use**:
- ✅ "Christian orthodox position"
- ✅ "Denominational considerations within Christianity"
- ✅ "Rejected by Christian orthodoxy because..."
- ✅ "Non-orthodox view from [group]"
- ❌ "All interpretations equally valid"
- ❌ Treating cults/other religions as valid Christian alternatives

**Scope**: Applies to all theological content - TBTA features, commentary, translation guidance

## AGENT behaviour
 - NEVER write notes or summaries to the home directory, instead create a plan in ./plans/{your-plan}/README.md then update it with results.  Keep the home directory clean.
 - When moving files always do a git commit before editing it so the git history is preserved
 - **Use subagents proactively from the start** - Don't wait until context is full. Delegate web research, file searches, experiments, and large tasks to subagents immediately to stay under 80% context. Focus on oversight, synthesis, and planning.
 - Prefer calling subagents in parallel, when possible 8 in parallel.
 - **Push after EVERY commit** - Run `git push -u origin <branch>` immediately after EVERY `git commit`. Never batch commits before pushing. Pattern: make change → commit → push → repeat.

### The Problem Being Solved

AI text prediction models tend to be more confident than accurate when dealing with Biblical texts. While they perform well on commonly cited passages (like John 3:16) in popular translations (NIV, KJV), accuracy degrades significantly with:
- Lesser-quoted texts (minor prophets, Numbers, etc.)
- Rare or minority language translations
- Verses that lack extensive Internet training data

### The Solution

By providing extensive contextual data (essentially "a book's worth of information") for every verse broken into smaller task focused files, AI systems can be grounded in truth rather than relying solely on compressed training data.

### Really important files

The following are key files you can load.  Don't load them automatically as it will fill your context, I've summarized the key points, read it for edge cases

 - STANDARDIZATION.md
   - 3 char ISO-639 language codes
   - USFM 3.0 for Bible book codes
   - pad numbers so they sort nicely
   - /commentary/{BOOK}/{chapter:03d}/{BOOK}.{chapter:03d}.{verse:03d}-{tool}.yaml for commentary files
   - /words/strongs/(G|H){strongs-number:04d}/{strongs-number}.strongs-{tool}.yaml for source language data
   - /topics/{lcc-code}/{slug}/{slug}-{tool}.yaml for topics
 - SCHEMA.md
   - cite sources inline with {source-id}: `{lang}-{version}` → `{lang}-{version}-{year}` for specificity
   - NEVER hallucinate: if from your memory use {llm-cs45}, if uncertain omit it
   - standard sections: source, translation, words, grammar, context, themes, cross_refs
   - loosely structured for easy merging/filtering across tools, use standard names so combines nicely on common structures, unstructured for unique values of the tool
 - REVIEW-GUIDELINES.md
   - 3 validation levels: critical (must pass), high priority (80%+), medium (60%+)
   - no fabrication, inline citations, no number predictions, data-file grounding only
   - new sources must be in ATTRIBUTION.md
 - ATTRIBUTION.md
   - all sources with copyright notices and citation codes
   - required for new sources: copyright, citation code, license type, purchase link

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [authenticwalk/mybibletoolbox-code](https://github.com/authenticwalk/mybibletoolbox-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
