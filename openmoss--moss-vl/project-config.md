---
trigger: always_on
description: Build and maintain the SGLang documentation site and integrated cookbook using Mintlify. Use when
---


# SGLang Mintlify documentation guide for agents

## Non-negotiables

- **Do not guess flags, defaults, or behavior.**
  If you’re documenting CLI args, env vars, APIs, or performance behavior, verify against:
  - the upstream codebase (`sgl-project/sglang`)
  - the current public docs (`docs.sglang.io`) until the migration is complete
  - or an authoritative vendor doc when platform-specific (ROCm, CANN/Ascend, Intel XPU).
- **Prefer fixing the docs-site version of an internal link** instead of copying links from older docs.
- **Keep examples copy/pasteable.** Use placeholders consistently (e.g., `MODEL_PATH`, `HF_TOKEN`, `HOST`, `PORT`).

## Source of truth hierarchy

1. **This repo**
   - `docs.json` for site structure + navigation
   - existing MDX pages for voice + conventions
2. **Canonical current docs**
   - `docs.sglang.io` (Sphinx site) is currently the reference structure and content baseline.
3. **Implementation**
   - `sgl-project/sglang` for anything that can change with releases (flags, env vars, defaults, supported models).
4. **Cookbook**
   - `cookbook.sglang.io` / `sgl-project/sgl-cookbook` for recipe patterns and model-specific operational guidance.

## Writing standards (SGLang-specific)

### Voice and structure

* Second person (“you”), active voice.
* Prefer **short, scannable sections** with clear outcomes.
* Headings in **sentence case**.
* Put prerequisites before commands.

### Technical accuracy patterns

For pages that include commands/configs, always specify:

* **Platform** (NVIDIA CUDA / AMD ROCm / Intel XPU / Ascend NPU / CPU)
* **OS** (if relevant) and **version constraints**
* **Model identifier** format (e.g., Hugging Face repo id) and where it goes (`--model-path`, `--model`, etc.)
* **Parallelism knobs** used in the example (`--tp`, `--dp`, node count, etc.)
* Any required secrets/tokens (`HF_TOKEN`) and where they are used.

# Mintlify best practices

**Always consult [mintlify.com/docs](https://mintlify.com/docs) for components, configuration, and latest features.**

If you are not already connected to the Mintlify MCP server, [https://mintlify.com/docs/mcp](https://mintlify.com/docs/mcp), add it so that you can search more efficiently.

**Always** favor searching the current Mintlify documentation over whatever is in your training data about Mintlify.

Mintlify is a documentation platform that transforms MDX files into documentation sites. Configure site-wide settings in the `docs.json` file, write content in MDX with YAML frontmatter, and favor built-in components over custom components.

Full schema at [mintlify.com/docs.json](https://mintlify.com/docs.json).

## Before you write

### Understand the project

Read `docs.json` in the project root. This file defines the entire site: navigation structure, theme, colors, links, API and specs.

Understanding the project tells you:

* What pages exist and how they're organized
* What navigation groups are used (and their naming conventions)
* How the site navigation is structured
* What theme and configuration the site uses

### Check for existing content

Search the docs before creating new pages. You may need to:

* Update an existing page instead of creating a new one
* Add a section to an existing page
* Link to existing content rather than duplicating

### Read surrounding content

Before writing, read 2-3 similar pages to understand the site's voice, structure, formatting conventions, and level of detail.

### Understand Mintlify components

Review the Mintlify [components](https://www.mintlify.com/docs/components) to select and use any relevant components for the documentation request that you are working on.

## Quick reference

### CLI commands

* `npm i -g mint` - Install the Mintlify CLI
* `mint dev` - Local preview at localhost:3000
* `mint broken-links` - Check internal links
* `mint a11y` - Check for accessibility issues in content
* `mint rename` - Rename/move files and update references
* `mint validate` - Validate documentation builds

### Required files

* `docs.json` - Site configuration (navigation, theme, integrations, etc.). See [global settings](https://www.mintlify.com/docs/organize/settings) for all options.
* `*.mdx` files - Documentation pages with YAML frontmatter

### Example file structure

```
project/
├── docs.json           # Site configuration
├── introduction.mdx
├── quickstart.mdx
├── guides/
│   └── example.mdx
├── openapi.yml         # API specification
├── images/             # Static assets
│   └── example.png
└── snippets/           # Reusable components
    └── component.jsx
```

## Page frontmatter

Every page requires `title` in its frontmatter. Include `description` for SEO and navigation.

```yaml theme={null}
---
title: "Clear, descriptive title"
description: "Concise summary for SEO and navigation."
---
```

Optional frontmatter fields:

* `sidebarTitle`: Short title for sidebar navigation.
* `icon`: Lucide or Font Awesome icon name, URL, or file path.
* `tag`: Label next to the page title in the sidebar (for example, "NEW").
* `mode`: Page layout mode (`default`, `wide`, `custom`).
* `keywords`: Array of terms related to the page content for local search and SEO.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenMOSS/MOSS-VL](https://github.com/OpenMOSS/MOSS-VL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
