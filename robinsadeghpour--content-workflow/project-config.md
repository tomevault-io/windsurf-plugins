---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Content Workflow**

A Claude Code-native content automation system that handles the full pipeline from idea discovery to scheduled publishing across LinkedIn, TikTok (EN + DE), and Instagram. Built with skills, agents, and cron automation to let Robin review and approve content in a single morning batch while the system handles research, drafting, media generation, and scheduling via Postiz.

**Core Value:** One morning session turns a curated idea backlog into platform-native content scheduled across all channels — no context-switching, no manual formatting, no copy-pasting between tools.

### Constraints

- **Tech stack**: Claude Code skills + agents as primary architecture, Node.js scripts where needed (Larry pipeline)
- **Publishing**: All scheduling through Postiz API
- **Language**: English master content, German localization for TikTok DE only
- **Human-in-the-loop**: No post publishes without Robin's explicit approval
- **Voice**: Must sound authentically Robin — LinkedIn slightly formal but real, TikTok/Instagram casual ("brooo", "bruh" energy)
- **Media priority**: Real visuals (logos, screenshots, product images) preferred over AI-generated imagery
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Node.js | 20+ LTS | Runtime for all scripts | Larry pipeline is already Node.js; Claude Code skills invoke bash which calls node scripts. Ecosystem depth for image, HTTP, file I/O is unmatched. |
| `@anthropic-ai/sdk` | 0.85.0 | Content generation, translation, humanizer calls | Native to the Claude Code environment. Used in-process for text generation, translation of EN→DE, voice calibration. Current latest as of research date. |
| `@postiz/node` | 1.0.8 | Scheduling + cross-platform publishing | Already integrated as a skill. Official SDK. Supports LinkedIn, TikTok, Instagram, 28+ platforms. Also exposes analytics API needed for feedback loop. Rate limit: 30 req/hr (non-posting). |
| `canvas` (node-canvas) | 2.x | Slide text overlay compositing | Larry 1.0.0 already ships this as `add-text-overlay.js`. Cairo-based, ~800K weekly downloads, longest track record, most compatible with browser Canvas2D API. The existing codebase uses `createCanvas` + `loadImage` from this package — no migration cost. |
| `sharp` | 0.34.5 | Image resize, format conversion, composite assembly | Fastest Node.js image processor (libvips). Used for final slide compositing, logo/screenshot scaling, format conversion (PNG→WebP for Instagram). Text is rendered via canvas, then composited via sharp for output. |
| `playwright` (Python CLI) | 1.x (Playwright sync) | HTML→slide screenshot pipeline | The existing linkedin skill's `screenshot-slides.py` already uses `playwright.sync_api`. This is the proven path: Claude writes HTML, Playwright renders and captures each `.slide` element at 2x DPR. Do not replace this. |
| `node-cron` | 4.2.1+ | Daily pulse cron trigger | 3M weekly downloads, minimal API, zero dependencies, full cron syntax. Used for `0 6 * * *` daily pulse and `0 20 * * *` end-of-day performance check. |
| `better-sqlite3` | latest | Idea backlog, performance history, voice profile cache | Fastest synchronous SQLite for Node.js. Single-user CLI context — sync API is ideal (no callback complexity). Outperforms lowdb by orders of magnitude once rows exceed ~1K. Stores: content ideas, post metadata, performance scores, scrape dedup hashes. |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `zx` | 8.x (`@lite` variant) | Shell orchestration in JS scripts | Wrapping multi-step pipeline calls (scrape → generate → overlay → schedule) in a readable async script without raw `child_process`. ~7x smaller than full zx. Use in pipeline runner scripts, not in skill SKILL.md files. |
| `cheerio` | 1.x | HTML parsing for web scraping | Scraping changelog pages, GitHub release pages, and static HTML content. No browser needed — use before reaching for Playwright/Puppeteer. |
| `node-fetch` / native `fetch` | Node 20 built-in | HTTP requests to APIs | Node 20 ships `fetch` natively. Do NOT install `node-fetch` unless targeting Node <18. Use native fetch for Postiz API, Apify actors, Supadata calls. |
| `@napi-rs/canvas` | latest | Alternative canvas if node-canvas install fails | Skia-based, pre-built binaries (no system lib deps like libcairo). 500K weekly downloads. Use as a drop-in fallback if node-canvas native build fails on a fresh machine. |
| `puppeteer` | 22.x | Screenshot fallback / Apify actor integration | Already present conceptually via Apify. Use Playwright for slide rendering (already implemented), Puppeteer only if an Apify actor explicitly requires it. |
| `openai` | latest | Image generation (gpt-image-1.5) | Larry already uses this for `generate-slides.js`. Keep as primary AI image gen. Use `gpt-image-1.5` (NEVER `gpt-image-1` per Larry's explicit warning). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [robinsadeghpour/content-workflow](https://github.com/robinsadeghpour/content-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
