---
trigger: always_on
description: <!-- markdownlint-disable-file -->
---

# Mendix Documentation Repository

<!-- markdownlint-disable-file -->

**Your role**: Edit and review Markdown documentation files under `content/en/docs/` following the style guidance and project-specific conventions below.

## Instruction Precedence

When instructions conflict, follow this order of precedence:

1. The user's current request.
2. Task-specific prompt files in `.github/prompts/*.prompt.md` (for Copilot) or skills in `.claude/skills/*/SKILL.md` (for Claude) when explicitly referenced or invoked.
3. Overlay instruction files (for example, `.github/release-notes-instructions.md`) when path-scoped.
4. This file (`CLAUDE.md`).
5. Mendix Style Guide files in `content/en/docs/community-tools/contribute-to-mendix-docs/style-guide/` for detailed grammar, terminology, and formatting rules.
6. Existing conventions in nearby pages within the same folder.
7. Microsoft Writing Style Guide.

### Critical Constraints

* Edit existing files in place using the Edit tool. Do not create new files unless explicitly requested.
* Preserve meaning and intent. Make the smallest set of edits that fully resolves the request.
* Never add product claims, technical behavior, ticket numbers, or release facts unless explicitly requested and sourced from provided content.
* Use Read to view files, Edit to modify them, Glob to find files by pattern, and Grep to search file contents.

### Autonomy Guidelines

Make these changes autonomously without asking:
* Grammar, spelling, and punctuation fixes
* Adding missing alt text to images
* Fixing URL formats (adding trailing slashes, converting to absolute paths)
* Standardizing terminology and capitalization
* Fixing broken link syntax
* Adding required front matter fields

Ask for confirmation before making these changes:
* Deleting substantial content (more than a sentence)
* Changing URLs or anchor IDs that may be referenced elsewhere (always use Grep to check for references first)
* Major structural reorganization or moving content between files
* Changing the meaning or technical accuracy of content
* Adding new sections, examples, or substantial content not explicitly requested
* Making changes that could affect functionality or user workflows

## Project Overview

* **What** – This repository contains the source code for the Mendix documentation site, which describes the Mendix low‑code application development platform. Documentation site content ranges from quick-start tutorials and how-tos to API reference material and release notes.
* **Who** – Target readers are developers, business analysts, system administrators, and partners who consume the docs for learning, troubleshooting, and reference. Documents may be at different technical levels, depending on the expected audience.
* **Tech stack** – Hugo-based static website based on the Docsy theme. Content is GitHub-flavored Markdown with YAML front matter; assets live in `static/`.

## Content Structure and Hierarchy

The canonical tree is **`/content/en/docs`**. Top‑level directories correspond to major product areas (e.g. `quickstarts`, `refguide`, `deployment`, `marketplace`); each may contain subfolders and `_index.md` files that define section landing pages.

Typical structure:

```
content/en/docs/
├── _index.md
├── quickstarts/
│   ├── _index.md
│   ├── hello-world.md
│   └── responsive-web-app.md
├── refguide/
│   ├── _index.md
│   ├── modeling/ …
│   └── runtime/ …
…
```

* **Index files (`_index.md`)** define landing pages or categories. They often use `cascade` to pass metadata to children and may set `type`, `layout`, `no_list`, `description_list`, etc.
* Other `.md` files represent individual articles, how‑tos, reference topics, release notes, etc. File names must be simple, lowercase, and hyphen‑separated.

Before creating a new file, use Glob to explore the directory structure and understand where the topic belongs.

## Style Standards

* **Guiding manual** – Mendix-specific style guides (see subsection below) extend and customize the Microsoft Writing Style Guide (https://learn.microsoft.com/style-guide/). Consult the Mendix style guides first for grammar, inclusive language, terminology, and formatting rules; use MSG as fallback for topics not covered by Mendix guides.
* **Tone** – Clear, concise, active voice; use imperative mood for procedures; second person (you/your) when addressing readers. Keep a conversational, straightforward tone. Present tense. Use American English and write for a global audience. Prefer short, everyday words; avoid or explain jargon. Keep it simple—short sentences and fragments are easier to scan and read, and prune excess words. Avoid marketing language.
* **Person** – Avoid first-person plural (we, us, our, let's) in all documentation except release notes.
* **Terminology** – Capitalize product names (Mendix Portal, Studio Pro, Team Server); use "microflow", "nanoflow", etc. consistently. Never use e.g. or i.e.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mendix/docs](https://github.com/mendix/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
