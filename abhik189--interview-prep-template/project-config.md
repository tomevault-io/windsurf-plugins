---
trigger: always_on
description: A personal interview prep wiki on a three-layer system (Karpathy's pattern):
---

# AGENTS.md — Interview Prep Wiki

## Purpose

A personal interview prep wiki on a three-layer system (Karpathy's pattern):

- **Layer 1 — `sources/`**: raw material you drop in (mock-interview recordings & transcripts, real-interview & career-chat notes, reading, project descriptions). Content is immutable — you own it, I never edit it. An unprocessed file lives directly in its source folder (e.g. `sources/projects/`); once I've mined it, I move it into that folder's `processed/` subfolder.
- **Layer 2 — `wiki/`**: LLM-maintained markdown synthesizing the sources into reusable answers, frameworks, and concepts. I (the LLM) own this entirely.
- **Layer 3 — this file**: how I operate the wiki.

The loop: you drop materials into `sources/`; I lift the durable content into `wiki/` and move the source into `processed/`; you query me and I answer from the wiki. On `lint` I flag gaps and stale claims. The wiki is the persistent, compounding artifact.

Targets: interviews for **[replace with your target functions — e.g. product management, GTM strategy & operations, engineering]** across **[N]** companies.

## Directory layout

```
sources/                      ← Layer 1 (raw material — content is immutable)
  recordings/                   audio/video of mock interviews       (drop unprocessed files here)
    processed/                    move here once mined
  transcripts/                  verbatim text from mock recordings   (drop unprocessed files here)
    processed/                    move here once mined
  interview-notes/              notes/recall of an interview or chat (drop unprocessed files here)
    processed/                    move here once mined
  reading/                      articles, book chapters, AI news     (drop unprocessed files here)
    processed/                    move here once mined
  projects/                     user's detailed project descriptions (drop unprocessed files here)
    processed/                    move here once mined

wiki/                         ← Layer 2 (you own)
  companies/<company>/
    overview.md                 strategy, products, recent moves
    <role>.md                   role-specific JD, role evaluation framework
  behavioral/<question>.md      one file per potential behavioral question
  frameworks/<name>.md          reusable analytical templates (GTM, AI transformation, prioritization, etc.)
  ai-knowledge/<name>.md        AI domain knowledge
  interview-debriefs/<role>/
  <round>-<date>-<role>.md      per-interview debriefs

AGENTS.md                     ← this file
index.md                      ← catalog of every wiki page
log.md                        ← chronological record of ingests, lints, structural changes
```

## Conventions

### Naming
- All file and folder names: lowercase, kebab-case (`tell-me-about-conflict.md`, not `Tell Me About Conflict.md`).
- Interview debriefs: `<round>-<YYYY-MM-DD>-<role>.md`, inside a per-role folder `wiki/interview-debriefs/<role>/`.
- Behavioral question slugs capture the **theme**, not the exact wording. Common phrasings live inside the file as a list. Example: `handling-conflict-with-peer.md` covers "tell me about a time you disagreed with a coworker," "describe a conflict you resolved at work," etc. When a new interviewer phrases the same theme differently, add their phrasing to the file's list — do not create a duplicate file.

### Cross-references
- Use `[[file-slug]]` wikilinks in markdown bodies. Link liberally.
- A `[[slug]]` that doesn't match an existing file is a TODO, not an error — it marks something worth writing later.
- Obsidian renders these natively.

### Frontmatter

Every wiki page starts with YAML frontmatter:

```yaml
---
name: file-slug
description: One-line summary
type: behavioral | framework | ai-knowledge | company-overview | company-role | interview-debrief
sources: [project-slug, transcript-slug]    # source files this draws from
related: [other-page-slug]                   # related wiki pages
updated: YYYY-MM-DD
---
```

Source files in `sources/` get lighter frontmatter:

```yaml
---
title: Source title
date-of-input: YYYY-MM-DD
type: recording | transcript | interview-notes | reading | project
---
```

### Behavioral Pages: 

 Focused on a behavioral concept that will be tested in interviews (e.g., handling tough stakeholders). It holds one example initially and accumulates more examples as projects are processed — it does not fork into near-duplicate pages.

**Single example** (the initial state of a freshly-created page):

```
## STAR — concise (~90s spoken)
## Expansion points
## How to improve this example further?
```

**Multiple examples** (after subsequent ingests):

```
## Choosing which example to use
[brief decision rule — e.g., which interviewer profile, which competency dimension each example illustrates best, which is the safer default]

## Primary example: [[project-slug]] — [one-line angle]
### STAR — concise (~90s spoken)
### Expansion points
### How to improve this example further?

## Secondary example: [[other-project-slug]] — [different angle]
### STAR — concise (~90s spoken)
### Expansion points
### How to improve this example further?

and so on

```

### Framework Pages


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbhiK189/interview-prep-template](https://github.com/AbhiK189/interview-prep-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
