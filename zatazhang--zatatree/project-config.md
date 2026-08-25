---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

> **Framework: Hugo** — This is a **Hugo** static site generator project (Hugo 0.141.0 extended). All content lives under `content/`, uses Hugo's TOML/YAML frontmatter conventions, and is rendered through the `hugo-theme-stack` theme. Use `hugo` / `hugo server` for build and dev work — do not introduce other static-site tooling.

## Repository Overview

This is a Hugo-based static blog repository named "ZataTree" that serves as a comprehensive knowledge base covering computer science, programming, AI/ML, and software development topics. The site is deployed at www.zata.cc and uses the hugo-theme-stack theme.

## Architecture

### Content Structure
The blog follows a hierarchical content organization:
- **Categories**: Top-level content classifications (Agent, Design, Engineering, DeepLearning, Grammar, Knowledge, Library, PaperReading, Platforms_Tools, Project_Application)
- **Tags**: Specific topics within categories
- **Posts**: Individual articles organized as `content/post/{category}/{tag}/{title}/index.md`

### Key Directories
- `content/post/`: Main content organized by category/tag/title structure
- `content/categories/`: Category metadata and images
- `content/tags/`: Tag metadata and images  
- `themes/hugo-theme-stack/`: Hugo theme (git submodule)
- `public/`: Generated static site output
- `static/`: Static assets (favicon, etc.)
- `assets/`: Theme assets and custom configurations

## Development Commands

### Hugo Site Management
```bash
# Local development with drafts
hugo server -D

# Local development without drafts  
hugo server

# Build static site
hugo --gc --minify

# Build with specific baseURL
hugo --gc --minify --baseURL "https://www.zata.cc/"
```

### Content Management Tool
The repository includes a custom Python tool (`zata.py`) for content management:

```bash
# Build the zata tool
pyinstaller --onefile --console --name=zata --clean zata.py

# Create new category
python zata.py create-category -c "CategoryName" -i "path/to/image.png"

# Create new tag  
python zata.py create-tag -c "CategoryName" -t "TagName" -i "path/to/image.png"

# Create new article
python zata.py create -c "CategoryName" -t "TagName" -b "Article Title"

# Search existing tags
python zata.py search-tags -k "keyword"

# Launch GUI interface
python zata.py gui
```

## Content Creation Workflow

### Article Structure
Each article must follow this structure:
```
content/post/{category}/{tag}/{title}/
├── index.md          # Main content file
├── images/           # Article images (optional)
└── {other-assets}    # Additional files (optional)
```

### Front Matter Template
```yaml
---
title: Article Title
description: Brief description
date: 2025-01-15T10:00:00+08:00
slug: title/index.md
image: images/index/index.png  # Optional
categories:
    - CategoryName
tags:
    - TagName
draft: false  # Set to true for drafts
---
```

### Important Conventions
- **Multilingual Support**: Default content language is zh-cn (Chinese), with English support
- **Image Handling**: Article images should be placed in `images/index/` subdirectory
- **Naming**: Use descriptive folder names that match the article title
- **Categories**: Must be pre-created using the zata tool
- **Tags**: Must be associated with categories and pre-created

### Cover Image Generation

Every published article must have a cover image referenced from its frontmatter as `image: images/index/index.<ext>`.

**When creating or reviewing an article**, follow this decision tree:

1. Check whether `images/index/index.png` (or `.svg`, `.jpg`, `.webp`) already exists in the article directory.
2. If it exists → leave it alone, just confirm the frontmatter `image:` field points to it.
3. If it is **missing**, do **not** search the web for a stock image. Instead, **generate an SVG cover**:
   - File path: `{article-dir}/images/index/index.svg`
   - Recommended canvas: **1200×630** (matches Open Graph / Twitter card aspect).
   - Style: minimal, geometric, with the article title and (optionally) 1–2 tag words. Use the `hugo-theme-stack` accent palette (primary `#5b87bf`, accent tones of blue / purple / teal). Avoid stock-photo clichés.
   - Format: inline SVG, no external assets, no remote fonts (use generic `sans-serif` / `serif`). Keep file size small (< 20 KB ideal).
4. After generating, add or update the frontmatter:
   ```yaml
   image: images/index/index.svg
   ```

This rule applies to any new article, any article extracted from another article's directory (e.g. the `记忆模块技术文档` extraction), and any article missing a cover during cleanup. SVG-first keeps the repo self-contained and avoids copyright issues.

## Deployment

### GitHub Actions
The repository uses GitHub Actions for automatic deployment:
- **Trigger**: Pushes to `hugo` branch
- **Hugo Version**: 0.141.0 (extended)
- **Target**: GitHub Pages
- **Workflow**: `.github/workflows/hugo.yaml`

### Manual Deployment
```bash
# Ensure git submodules are updated
git submodule update --init --recursive

# Build and deploy
hugo --gc --minify --baseURL "https://www.zata.cc/"
```

## Theme Configuration

### Key Settings in hugo.yaml
- **Theme**: hugo-theme-stack

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZataZhang/ZataTree](https://github.com/ZataZhang/ZataTree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
