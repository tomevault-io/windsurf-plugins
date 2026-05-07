---
trigger: always_on
description: > This file is the **single source of truth** for any LLM operating on this vault.
---

# Second Brain — Agent Schema

> This file is the **single source of truth** for any LLM operating on this vault.
> Read this FIRST before making any changes.

## Purpose

This Obsidian vault is an **AI-managed knowledge base** following the Karpathy LLM Knowledge Base methodology. The LLM writes and maintains all wiki content. The human rarely edits directly — they ingest raw data, ask questions, and review outputs.

## Directory Structure

```
Second-brain/
├── raw/                 ← Source documents. NEVER modify, only ADD.
│   ├── articles/        ← Web articles (clipped or pasted)
│   ├── papers/          ← Academic papers, PDF notes
│   ├── repos/           ← GitHub repo notes, README summaries
│   ├── videos/          ← YouTube transcripts, video notes
│   ├── tweets/          ← X/Twitter threads, social posts
│   └── misc/            ← Images, CSV, datasets, other
│
├── wiki/                ← Compiled knowledge. AI-maintained.
│   ├── _index.md        ← Master index (MUST stay updated)
│   ├── _glossary.md     ← Key terms and definitions
│   ├── concepts/        ← Concept articles (the core)
│   ├── tools/           ← Tool/product evaluations
│   ├── people/          ← Notable people profiles
│   └── comparisons/     ← A vs B analysis articles
│
├── outputs/             ← Query results, generated content
│   ├── reports/         ← Deep-dive research reports
│   ├── slides/          ← Marp-format slide decks
│   ├── charts/          ← Generated visualizations
│   └── summaries/       ← Quick summaries on demand
│
├── sessions/            ← Session logs TẬP TRUNG từ tất cả dự án
│   ├── hermes-agent/    ← Sessions từ Hermes-Agent
│   ├── kenh-youtube/    ← Sessions từ kenh-youtube
│   ├── bai-viet-mxh/    ← Sessions từ bai-viet-mxh
│   ├── rag-notebooklm/  ← Sessions từ RAG-notebooklm
│   └── second-brain/    ← Sessions từ Second-brain
│
└── AGENTS.md            ← THIS FILE — vault schema
```

## File Conventions

### Naming
- **kebab-case**: `retrieval-augmented-generation.md`
- **No spaces**: Use hyphens
- **Descriptive**: Name should convey content at a glance

### Frontmatter (YAML) — Required for ALL files

```yaml
---
title: "Human-readable title"
sources: ["[[raw/source-1.md]]", "[[raw/source-2.md]]"]  
date_added: 2026-04-03
tags: [concept, ai, rag]
aliases: [alias1, alias2, tên tiếng Việt]
status: draft | reviewed | canonical
related:
  - "[[other-article]]"
  - "[[another-one]]"
summary: "One-line summary for _index.md"
---
```

### Content Rules
- Use `[[wikilinks]]` for internal links (Obsidian-native)
- Images: Store in same directory as the .md referencing them
- Code blocks: Always specify language (```python, ```bash, etc.)
- Headers: Start at ## (h2) inside articles (h1 = title in frontmatter)

### Writing Tone — Bách Khoa Toàn Thư
- Viết giọng văn trung lập, dẫn chứng cụ thể. Không phải blog, không phải notes.
- Tránh: "thú vị là", "đáng chú ý", "rất quan trọng", "groundbreaking", "legendary"
- Tránh editorial voice: "interestingly", "importantly", "it should be noted"
- Cấu trúc bài theo chủ đề (thematic), không theo dòng thời gian (chronological)
- Cảm xúc/nhận định truyền qua direct quotes từ raw source
- Ưu tiên quotes đắt giá, tránh quote tràn lan
- 1 ý = 1 câu. Câu ngắn. Viết đoạn văn, hạn chế bullet-point trừ khi liệt kê.
- Attribution thay vì assertion: "Karpathy mô tả nó là..." thay vì "Nó rất..."

## Index Maintenance

The file `wiki/_index.md` is the **master catalog**. Rules:
1. MUST list every article in `wiki/` with one-line summary
2. Group by subdirectory (concepts, tools, people, comparisons)
3. Update `_index.md` EVERY time a wiki article is added/removed
4. Include article count and last-updated timestamp

## Compilation Rules

When compiling from `raw/` to `wiki/`:
1. Read the raw source completely
2. Check `wiki/_absorb_log.json` để xem raw nào đã compile
3. Identify key concepts, tools, and people mentioned
4. For each:
   - Check if wiki article already exists → UPDATE
   - If not → CREATE new article
5. **Re-read toàn bộ bài wiki TRƯỚC KHI cập nhật — non-negotiable**
6. Sau khi đọc lại, tự hỏi: "Entry mới bổ sung chiều sâu gì mà bài chưa có?"
   - Nếu câu trả lời là "không gì mới" → KHÔNG sửa bài
7. **Contradiction Check** — Trước khi ghi đè bất kỳ thông tin nào, kiểm tra xem claim mới có mâu thuẫn với claim cũ không. Nếu mâu thuẫn → KHÔNG ghi đè, thêm callout `[!warning] Mâu Thuẫn Chưa Giải Quyết` và tag `needs-review`.
8. Khi cập nhật, **integrate** nội dung mới vào mạch viết hiện có — không chỉ append bullet point ở cuối
9. Add `[[backlinks]]` to related existing articles
10. Update `_index.md`
11. Update `wiki/_absorb_log.json` — ghi nhận raw đã compile
12. Never delete content from existing articles — refine and integrate

## Quality Standards

- Each wiki article should be **self-contained** (understandable alone)
- Minimum 200 words per concept article
- Each article should link to ≥2 other wiki articles
- Avoid duplicating content — link instead
- Use Vietnamese for content, English for technical terms

### Article Size Guardrails
- **Giới hạn:** 15–120 dòng nội dung (không tính frontmatter)
  - Dưới 15 dòng → tag `status: stub`, ưu tiên bổ sung khi có raw mới
  - Trên 120 dòng → xem xét tách sub-topic thành bài riêng

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KHOAAI-HILL/llm-wiki-template](https://github.com/KHOAAI-HILL/llm-wiki-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
