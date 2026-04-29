---
trigger: always_on
description: This file provides guidance to Qoder (qoder.com) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Qoder (qoder.com) when working with code in this repository.

## Project Overview

**DeepTalk** - Markdown to HTML converter and LLM-powered article generator for WeChat public accounts.

Features:
1. Convert Markdown to styled HTML with multiple themes
2. Render for WeChat public account (inline CSS)
3. Preview Markdown/HTML in browser
4. Generate articles using LLM (OpenAI/DeepSeek)
5. Publish to WeChat draft box via API

Works with CLI tools like qoder, opencode, claude, codex, gemini.

## Commands

```bash
# Install dependencies
npm install

# Build TypeScript to dist/
npm run build

# Lint
npm run lint

# === Markdown to HTML ===

# Convert Markdown to HTML
npm run md2html -- -i input.md -o ./contents

# Preview Markdown
npm run preview-md -- -i input.md

# Preview HTML directory
npm run preview-html -- -i ./contents

# === WeChat Public Account ===

# Convert to WeChat format (inline CSS)
npm run wechat -- -i article.md --copy

# Publish to WeChat draft box
npm run publish -- -i article.md --title "标题"

# List drafts
npm run drafts

# Fetch WeChat article (save as Markdown)
npm run fetch-wechat -- -u "https://mp.weixin.qq.com/s/xxx"

# === Web Interface ===

# Start web UI
npm run web

# === LLM Generation ===

# Generate article (requires API key)
npm run generate -- -t "topic" -c [tech|ai|invest]

# Generate article and save to contents directory
npm run generate -- -t "topic" -c tech --contents
```

## Configuration

Required environment variables (set in `.env`):

```env
# LLM API (for article generation)
OPENAI_API_KEY=your-api-key
# or
DEEPSEEK_API_KEY=your-key
# Optional: custom base URL (defaults to https://api.openai.com/v1)
OPENAI_BASE_URL=https://api.openai.com/v1
DEEPSEEK_BASE_URL=https://api.deepseek.com/v1

# WeChat Public Account (for publishing)
WECHAT_APP_ID=wx...
WECHAT_APP_SECRET=...

# Unsplash (optional, for auto image search in articles)
UNSPLASH_ACCESS_KEY=...
```

## Architecture

```
src/
├── index.ts               # CLI entry (Commander.js)
├── types.ts                # Top-level type definitions (Category, ArticleConfig, Article)
├── skills/
│   ├── index.ts            # Barrel export
│   ├── htmlGenerator.ts    # Standard HTML renderer
│   ├── wechatRenderer.ts   # WeChat renderer (inline CSS)
│   └── types.ts            # Theme & render type definitions
├── wechat/
│   ├── index.ts            # Barrel export
│   └── api.ts              # WeChat API client
├── generators/
│   └── index.ts            # LLM article generation (OpenAI)
├── prompts/
│   └── index.ts            # Category-specific system prompts
├── templates/
│   └── index.ts            # Article structure templates & auto-detection
├── web/
│   ├── index.ts            # Barrel export
│   └── server.ts           # Express web server & inline HTML pages
├── config/
│   └── index.ts            # API configuration (dotenv)
└── utils/
    ├── index.ts            # Barrel export, saveArticle(), formatDate()
    └── fetcher.ts          # WeChat article fetcher

.agents/skills/
├── wechat-article/         # WeChat article generation skill
│   ├── SKILL.md            # Skill definition (themes, templates, image handling)
│   ├── HTML-CSS-SPEC.md   # HTML/CSS technical spec for WeChat
│   ├── design-system/
│   │   └── references/    # Per-theme DESIGN.md specs
│   │       ├── tech/
│   │       ├── business/
│   │       ├── claude/
│   │       ├── minimal/
│   │       ├── dark-finance/
│   │       ├── standard/
│   │       ├── neo-brutalist/
│   │       ├── luxury-editorial/
│   │       ├── japanese-zen/
│   │       └── vintage-newspaper/
│   └── templates/          # Example HTML templates
├── wechat-fetcher/         # WeChat article fetcher skill
│   ├── SKILL.md
│   ├── config.json
│   ├── README.md
│   └── examples/
├── wechat-publisher/       # WeChat draft publisher skill
│   └── SKILL.md
└── humanizer-zh/           # AI writing trace remover (Chinese)
    ├── SKILL.md
    └── LICENSE

contents/              # Default content storage
output/                # LLM output directory (article list data source)
markdowns/             # Markdown source files (editable via /editor)
public/                # Static assets (logo.svg etc.)
```

## Key Modules

### types.ts

Top-level type definitions:
- `Category` - `'tech' | 'ai' | 'invest'`
- `TemplateType` - `'tutorial' | 'analysis' | 'news' | 'story' | 'listicle' | 'review' | 'auto'`
- `ThemeType` - `'tech' | 'business' | 'claude' | 'minimal' | 'auto'`
- `ArticleConfig` - LLM generation input config
- `Article` - LLM generation output structure

### skills/types.ts

Renderer-specific type definitions:
- `Theme` - `'tech' | 'minimal' | 'business'`
- `WeChatTheme` - `'tech' | 'business' | 'minimal'`
- `HtmlArticleConfig` - `generateHtml()` input config
- `WeChatRenderOptions` - `renderForWeChat()` options
- `WeChatArticle` - WeChat article structure

### web/server.ts

Express web server providing:
- `GET /` - Article management dashboard (shows files from output/)
- `GET /editor` - Markdown editor with file management
- `GET /api/files` - List HTML/Markdown files from output/
- `GET /api/files/:id` - Retrieve or render a file
- `POST /api/render` - Convert Markdown to HTML preview

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [honwhy/DeepTalk](https://github.com/honwhy/DeepTalk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
