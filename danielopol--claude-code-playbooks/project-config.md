---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Playbooks is a Next.js 16 website that hosts copy-paste workflows for Claude Code users. It's a content-driven site where playbooks are authored as MDX files with frontmatter metadata.

## Commands

```bash
npm run dev     # Start development server on http://localhost:3000
npm run build   # Production build
npm run lint    # Run ESLint
```

## Architecture

### Content System
- Playbooks are MDX files in `src/content/playbooks/`
- Each playbook has YAML frontmatter defining: title, description, seoHook, targetAudience, exampleUseCase, category, difficulty, timeToSetup, author, tags, createdAt
- `src/lib/playbooks.ts` reads MDX files at build time using `gray-matter` and extracts CLAUDE.md templates from code blocks
- Categories are defined in `src/lib/categories.ts` with 9 categories across 7 verticals

### Type System
- `src/types/playbook.ts` defines `Playbook`, `PlaybookFrontmatter`, `Category` (union type), and `Difficulty` types
- Category type must match one of the predefined category IDs

### Routing (App Router)
- `/` - Homepage with featured playbooks and category grid
- `/playbooks` - Browse all playbooks with search/filter
- `/playbooks/[slug]` - Individual playbook page (static generation via `generateStaticParams`)
- `/categories/[category]` - Filtered playbooks by category
- `/api/subscribe` - Newsletter subscription endpoint

### UI Components
- Uses shadcn/ui (new-york style) with components in `src/components/ui/`
- Tailwind CSS v4 with CSS variables for theming (oklch colors)
- Custom components: PlaybookCard, CategoryBadge, DifficultyBadge, CopyButton, SearchBar, Newsletter

### Path Aliases
- `@/*` maps to `./src/*` (configured in tsconfig.json)

## Adding a New Playbook

### 1. Create the template file

Create `public/templates/{slug}.md` with the raw CLAUDE.md template content — no frontmatter, no wrapper, just the template text that users will download.

### 2. Create the MDX file

Create `src/content/playbooks/{slug}.mdx` where `{slug}` matches the template filename:

```mdx
---
title: "Playbook Title"
description: "Short description"
seoHook: "A compelling 1-2 sentence hook explaining the problem this playbook solves — written to engage the reader emotionally or practically."
targetAudience: "persona1, persona2, persona3"
exampleUseCase: "\"Example user prompt\" → What the playbook produces in response"
category: "file-organization"  # Must match a Category type
difficulty: "beginner"         # beginner | intermediate | advanced
timeToSetup: "5 minutes"
author: "community"
sourceUrl: "https://..."       # URL where the playbook content originated (if provided)
tags: ["tag1", "tag2"]
createdAt: "2026-01-09"
---

## What This Does

Brief explanation of what this playbook helps the user accomplish.

---

## Quick Start

### Step 1: Create a Project Folder

...setup instructions...

### Step 2: Download the Template

Click **Download** above, then move the file...

### Step 3: Start Working

...how to start using it...
```

### Required: SEO Hook, Target Audience & Example Use Case

Every playbook **must** include these three frontmatter fields. They render as a prominent section on the playbook page (between the tags and the CLAUDE.md Template download):

- **`seoHook`**: A compelling 1-2 sentence hook that describes the problem this playbook solves. Write it to resonate emotionally or practically with the target user. Example: `"Finding the right papers in a sea of millions of publications is like searching for needles in a haystack — and missing one key study can undermine your entire thesis or research project."`
- **`targetAudience`**: A comma-separated list of personas who benefit from this playbook. Displayed as "**Who it's for:** ...". Example: `"graduate students, research assistants, professors, science journalists, systematic reviewers"`
- **`exampleUseCase`**: A concrete before → after example showing what a user might ask and what the playbook produces. Use the format `"user prompt" → result description`. Example: `"\"Find recent papers on transformer architectures for protein folding\" → Curated list of 15 high-impact papers with relevance scores, methodology summaries, and key findings synthesized"`

### 3. Enrich the MDX with template insights

After creating the MDX and template files, scan the template (`public/templates/{slug}.md`) for useful contextual sections and add them to the MDX content. Look for:

- **Tips** and **best practices** (e.g., "Tips for Best Results", "Pro Tips")
- **Limitations** and **caveats** (e.g., "Known Limitations", "Important Notes")
- **Examples** (e.g., example commands, sample inputs/outputs, use cases)
- **Supported formats/platforms** (e.g., "Supported Languages", "Compatible Formats")
- **Prerequisites details** (e.g., required tools with install commands, API keys needed)
- **Troubleshooting** guidance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Danielopol/Claude-Code-Playbooks](https://github.com/Danielopol/Claude-Code-Playbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
