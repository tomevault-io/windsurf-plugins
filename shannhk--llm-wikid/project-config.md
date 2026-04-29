---
trigger: always_on
description: Personal knowledge base maintained by an LLM agent.
---

# LLM Knowledge Base - Schema

## Overview

Personal knowledge base maintained by an LLM agent.
Raw sources live in `raw/`. The compiled wiki lives in `wiki/`.
You (the AI) maintain all wiki content. The user directs strategy and source
collection; you execute compilation, maintenance, and queries.

This is NOT RAG. Knowledge is pre-compiled into wiki pages. The agent reads
structured pages, not raw chunks. Every query compounds the system.

## Directory Structure

### Wiki

- `wiki/index.md` - Master index linking every page with TLDR for fast scanning
- `wiki/log.md` - Append-only changelog of all operations
- `wiki/concepts/` - Concept pages (ideas, frameworks, topics)
- `wiki/entities/` - Entity pages (people, companies, tools, protocols)
- `wiki/sources/` - Source summary pages (one per raw source)
- `wiki/syntheses/` - Cross-cutting analysis across sources
- `wiki/outputs/` - Filed answers to queries
- `wiki/sops/` - Repeatable processes and workflows
- `templates/` - Starter templates per page type (for Obsidian Templater)

### Raw

- `raw/ideas/` - Idea notes and brainstorms
- `raw/bookmarks/` - Bookmarked tweet and article sources
- `raw/x-archive/` - X archive tweets (exported from X/Twitter)
- `raw/clippings/` - Obsidian Web Clipper inbox. Everything lands here. During INGEST, files get sorted to the correct subfolder and removed from clippings.
- `raw/articles/` - Long-form articles and blog posts (sorted from clippings)
- `raw/papers/` - Research papers and PDFs (sorted from clippings)
- `raw/assets/` - Images, videos, and other media

New wiki pages go in the subfolder matching their `type` frontmatter field. Wikilinks use filename only (Obsidian resolves via shortest-path setting).

## Frontmatter Schema

Every wiki page MUST have YAML frontmatter:

```yaml
---
title: "Page Title"
tldr: "One to two sentence TLDR - used for index scanning to save tokens"
date_created: YYYY-MM-DD
date_modified: YYYY-MM-DD
type: concept | entity | source | project | sop | synthesis | output
tags: [topic-tag, another-tag]
sources: ["[[source-page]]"]
original_url: "https://..." # For source pages - link to the original content
explored: false # Validation gate. Only a human sets this to true. AI always sets false.
confidence: high | medium | low | uncertain # How confident is the content? Be honest.
---
```

### Validation Gate

`explored` defaults to **false** on every AI-created or AI-updated page. Only the user can set it to `true` after reviewing the content. This prevents blind trust in AI-generated knowledge. The agent must never set `explored: true`.

`confidence` signals how well-supported the content is:
- **high** - multiple corroborating sources, well-established topic
- **medium** - single source or limited evidence, reasonable synthesis
- **low** - speculative, early-stage idea, thin evidence
- **uncertain** - contradictory sources or insufficient information

## Page Types

| Type | What it is |
|------|-----------|
| concept | An idea, framework, or topic |
| entity | Person, company, tool, protocol |
| source | Summary of one raw source |
| project | Active project with goals and context |
| sop | Repeatable process or workflow |
| synthesis | Cross-cutting analysis across sources |
| output | Filed answer to a query |

## Source Resolution (Resolve Before Classify)

`raw/` is a messy inbox. Before classification, INGEST must resolve each input to its full content using the right tool.

### Resolution Tool Mapping

| Input Pattern | Tool | What It Extracts |
|--------------|------|-----------------|
| YouTube URL (youtube.com, youtu.be) | `yt-dlp` | Transcript (auto-subs), title, description, metadata |
| X/Twitter URL (x.com, twitter.com) | X API (OAuth) | Tweet text, thread, author, metrics. If video/image attached, also `yt-dlp` for video and download images |
| Web URL / Reddit URL | `scrapling` | Page content, title, author, date |
| PDF file (.pdf) | Read directly / `summarize` CLI | Full text extraction |
| Bookmark dump (list of URLs) | Parse each URL, resolve individually | One resolved file per URL |
| Plain text / notes / ideas | Read as-is | No resolution needed |

### Media Extraction

Every source with attached media MUST have that media downloaded and analyzed:

**Images:**
- Download to `raw/assets/images/` with descriptive kebab-case names
- Analyze the image content (diagrams, screenshots, infographics, memes, charts)
- Include analysis in the source summary: what the image shows, key data points, visual format
- Reference in the raw file frontmatter: `media: ["raw/assets/images/filename.png"]`
- Embed in wiki source pages: `![[raw/assets/images/filename.png]]`

**Videos:**
- Extract transcript via `yt-dlp --write-auto-sub --skip-download`
- Download thumbnail to `raw/assets/images/`
- If short video (under 2 min), describe the visual content in the source summary
- Reference in raw file frontmatter: `media_transcript: "raw/assets/filename-transcript.txt"`

Images often contain the actual information (diagrams, frameworks, data). A tweet saying "here's my framework" with an image of the framework is useless without the image. The wiki must capture the visual knowledge, not just the text.

Resolution updates the file in `raw/` in-place, preserving the original URL in frontmatter:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shannhk/llm-wikid](https://github.com/shannhk/llm-wikid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
