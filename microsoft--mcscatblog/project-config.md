---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

This is the Microsoft Copilot Studio CAT team's technical blog, hosted at https://microsoft.github.io/mcscatblog/. It uses Jekyll with the Chirpy theme for static site generation on GitHub Pages.

## Commands

```bash
# Install dependencies
bundle install

# Run local development server (preferred)
./tools/run.sh

# Build the site
bundle exec jekyll build
```

## Writing Posts

Posts go in `_posts/` with filename format: `YYYY-MM-DD-post-slug.md`

### Front Matter Template

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
categories: [category1, category2]
tags: [tag1, tag2]
description: Brief description for SEO
author: authorkey  # Must match a key in _data/authors.yml
image:
  path: /assets/posts/post-slug/header-image.png
  no_bg: true  # Optional
published: true  # Set to false for drafts
---
```

### Author Setup

Add new authors to `_data/authors.yml` with:
- `name`: Display name
- `email`: Optional
- `avatar`: GitHub avatar URL (e.g., `https://github.com/username.png`)
- `url`: Optional link to profile/website

### Assets

Store post images in `assets/posts/<post-slug>/`. Reference in posts as `/assets/posts/<post-slug>/image.png`.

### Formatting Reference

See Chirpy documentation for advanced formatting: https://chirpy.cotes.page/posts/write-a-new-post/

## Project Structure

- `_posts/` - Blog posts in Markdown
- `_data/authors.yml` - Author definitions
- `_tabs/` - Navigation pages (about, archives, categories, tags)
- `_config.yml` - Site configuration (theme, SEO, analytics, comments)
- `assets/` - Static files (images, post assets)

## Blog Post Review

Review instructions are in `.github/instructions/posts.instructions.md`. Reviews prioritize: (1) narrative and structure, (2) technical accuracy, (3) reader experience, (4) front matter, (5) scope. No percentage scoring — output is a ranked list of 5-7 issues with quoted text and suggested fixes.

## Important Notes

- **Author GitHub usernames:** To identify PR authors, check `_data/authors.yml` which maps author keys to names and GitHub usernames (e.g., CATDAB = Doug Bellingeri)

---

## Adilei Writing Style Guide

This section captures adilei's writing patterns for Claude to replicate when drafting posts.

> **Post-writing step:** After completing each post, update this style guide with any corrections, terminology changes, or stylistic feedback provided during the writing session.
{: .prompt-info}

### Quick Reference Checklist

- [ ] Opens with real-world problem or relatable scenario
- [ ] Conversational tone with contractions ("you've", "it's", "don't")
- [ ] Direct address to reader ("you", "we", "let's")
- [ ] Complete working code examples (not snippets)
- [ ] Progressive disclosure: simple → detailed → complete
- [ ] Explicit trade-offs and limitations acknowledged
- [ ] Uses prompt boxes (`.prompt-tip`, `.prompt-warning`) for callouts
- [ ] Header image in front matter
- [ ] Ends with engagement question for comments
- [ ] 5-8 specific tags in front matter
- [ ] 2-3 internal links to related posts (SEO requirement)
- [ ] Tags tuned so Chirpy's auto-generated "Further Reading" shows relevant posts

### Voice & Tone

**Personality:**
- Conversational and approachable with occasional humor
- Friendly asides: "Bear with me for a sec", "I hear you say"
- Self-aware and transparent about limitations
- Pop culture references welcome (D&D, LOTR, Transformers, etc.)

**Punctuation:** Be frugal with em-dashes. Prefer commas or restructuring sentences over em-dashes.

**Formality:** Semi-formal balance—professional but not academic. Examples:
- Good: "You've probably run into this before..."
- Good: "Okay, I made that last one up, but it *should* be true"
- Avoid: "One must consider the implications of..."
- Avoid: "It is imperative that developers understand..."

**Rhetorical Techniques:**
- Myth-busting: "Myth vs. Reality" sections for misconceptions
- Comparative tables showing features/trade-offs
- Direct questions: "But what if you need X?"
- Strategic reveals: "But here's a little-known capability..."

### Terminology & Sensitivity

**Copilot Studio terminology:**
- Use "agents" not "bots"
- "Conversation Start topic" not "Greeting topic"
- In code comments, say "mocked" not "fake"

**Direct Line / WebChat accuracy:**
- A Direct Line conversation exists (has a conversation ID) as soon as connectivity is established, even before any topics are triggered
- `DIRECT_LINE/INCOMING_ACTIVITY` is client-side injection into WebChat's incoming message stream, not messages "coming from" the agent
- Be precise: "no topics triggered" is different from "no conversation started"

**Cost/licensing sensitivity:**
- Don't be explicit about cost savings, Copilot credits, or billing implications
- Imply rather than state directly: "someone eventually notices" vs "you get charged"
- Avoid promising savings—let readers draw their own conclusions
- Keep financial references tongue-in-cheek, not preachy

**Tone about users:**
- Don't be derogatory about visitor behavior (e.g., avoid "closes tab without engaging")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/mcscatblog](https://github.com/microsoft/mcscatblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
