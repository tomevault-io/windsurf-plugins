---
trigger: always_on
description: This file provides comprehensive guidance for AI coding agents working with the Skill Seekers Configs repository.
---

# AGENTS.md

This file provides comprehensive guidance for AI coding agents working with the Skill Seekers Configs repository.

## Project Overview

**Skill Seekers Configs** is a community repository of JSON configuration files for the [Skill Seekers](https://github.com/yusufkaraaslan/Skill_Seekers) tool. These configs tell Skill Seekers how to scrape documentation websites and/or GitHub repositories to generate AI skills compatible with Claude AI.

**Key Characteristics:**
- This repository contains **no source code**, **no build system**, and **no tests**
- Work here is purely authoring, validating, and maintaining JSON configuration files
- Configs are consumed by the external `skill-seekers` Python CLI tool
- The repository serves as a central registry that Skill Seekers can fetch configs from via API

## Repository Structure

```
skill-seekers-configs/
├── official/                    # Verified, production-ready configs (178 total)
│   ├── ai-ml/                  # 34 AI/ML tools
│   ├── api-tech/               # 2 API technologies
│   ├── build-tools/            # 7 build tools
│   ├── cloud/                  # 9 cloud platforms
│   ├── cms/                    # 3 CMS platforms
│   ├── css-frameworks/         # 6 CSS frameworks
│   ├── data-science/           # 4 data science tools
│   ├── databases/              # 16 databases & ORMs
│   ├── development-tools/      # 8 dev tools
│   ├── devops/                 # 9 DevOps tools
│   ├── game-engines/           # 35 game engines
│   ├── gaming/                 # 1 gaming platform
│   ├── graphics/               # 2 graphics APIs
│   ├── languages/              # 1 language
│   ├── messaging/              # 2 messaging systems
│   ├── mobile/                 # 2 mobile frameworks
│   ├── payments/               # 1 payment platform
│   ├── search/                 # 1 search engine
│   ├── security/               # 3 auth/security tools
│   ├── testing/                # 11 testing tools
│   ├── web-frameworks/         # 21 web frameworks
│   └── test-examples/          # 4 templates & test configs
├── community/                   # Community-submitted configs pending review
│   └── .gitkeep                # Empty directory marker
└── .github/
    └── ISSUE_TEMPLATE/         # GitHub issue templates for config submissions
```

## Configuration File Formats

### 1. Simple Format (Single-Source Documentation Scraping)

Used for scraping a single documentation website.

```json
{
  "name": "framework-name",
  "description": "When to use this skill...",
  "base_url": "https://docs.example.com/",
  "start_urls": [
    "https://docs.example.com/getting-started",
    "https://docs.example.com/api"
  ],
  "selectors": {
    "main_content": "article, main, div[role='main']",
    "title": "h1, title",
    "code_blocks": "pre code, pre"
  },
  "url_patterns": {
    "include": ["/docs/", "/guide/", "/api/"],
    "exclude": ["/blog/", "/search/", "/_static/"]
  },
  "categories": {
    "getting_started": ["introduction", "getting-started", "quick", "setup"],
    "core_concepts": ["concept", "fundamental", "architecture"],
    "api_reference": ["api", "reference", "method", "function"]
  },
  "rate_limit": 0.5,
  "max_pages": 300
}
```

### 2. Unified Format (Multi-Source: Documentation + GitHub)

Used for combining documentation scraping with GitHub repository analysis (C3.x codebase analysis features).

```json
{
  "name": "framework-name",
  "description": "Complete framework knowledge combining docs and codebase...",
  "merge_mode": "rule-based",
  "sources": [
    {
      "type": "documentation",
      "base_url": "https://docs.example.com/",
      "extract_api": true,
      "selectors": {
        "main_content": "article",
        "title": "h1",
        "code_blocks": "pre code"
      },
      "url_patterns": {
        "include": ["/docs/"],
        "exclude": ["/blog/"]
      },
      "categories": {
        "getting_started": ["intro", "quickstart"],
        "api": ["api", "reference"]
      },
      "rate_limit": 0.5
    },
    {
      "type": "github",
      "repo": "owner/repo",
      "enable_codebase_analysis": true,
      "code_analysis_depth": "deep",
      "fetch_issues": true,
      "max_issues": 100,
      "fetch_changelog": true,
      "fetch_releases": true,
      "file_patterns": [
        "src/**/*.py",
        "lib/**/*.py"
      ]
    }
  ]
}
```

### 3. PDF Extraction Format

For extracting content from PDF files.

```json
{
  "name": "example_manual",
  "description": "PDF extraction configuration...",
  "pdf_path": "docs/manual.pdf",
  "extract_options": {
    "chunk_size": 10,
    "min_quality": 5.0,
    "extract_images": true,
    "min_image_size": 100
  },
  "categories": {
    "getting_started": ["introduction", "setup"]
  }
}
```

## Configuration Schema Reference

### Common Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Config identifier (lowercase, hyphen-separated) |
| `description` | string | Yes | Explains WHEN to use the skill, lists capabilities |
| `base_url` | string | Simple only | Base URL of documentation site |
| `start_urls` | array | No | Entry points for crawling (defaults to base_url) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yusufkaraaslan/skill-seekers-configs](https://github.com/yusufkaraaslan/skill-seekers-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
