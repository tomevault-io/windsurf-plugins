---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**easy-amazon-voc** is an AI-powered comment analysis platform that helps product teams discover user pain points and needs from customer reviews. It uses a four-dimensional value model to generate multi-dimensional tag analysis and user personas.

**Core Transformation**: This project has been converted from a social media crawling + pain point analysis system to a CSV upload + AI comment tagging system (based on easy-amazon-voc methodology).

## Development Commands

```bash
# Install dependencies
npm install

# Development server with Turbopack
npm run dev

# Production build
npm run build

# Run production server
npm run start

# Lint code
npm run lint
```

## Architecture Overview

### System Flow

```
1. User uploads CSV file → 2. File validation & parsing →
3. AI generates tag system (Step 1) → 4. AI tags each review (Step 2) →
5. Word frequency calculation → 6. User persona generation →
7. Generate processed CSV for download
```

### Frontend (Next.js 15 App Router)
- `src/app/[locale]/page.tsx` - Main comment analysis page
- `src/app/api/comment-analysis/` - Comment analysis API endpoints
- `src/components/` - React components (UI)

### Backend Services (`lib/services/`)
- `comment-analysis-service.ts` - **Core**: AI two-step analysis (tag system generation + review tagging)
- `comment-job-manager.ts` - **Core**: Async job management with progress tracking
- `word-frequency-service.ts` - Tag frequency statistics

### Data Processing (`lib/utils/`)
- `csv-parser.ts` - CSV file parsing with encoding detection
- `csv-generator.ts` - Generate processed CSV with tags

### AI Prompts (`lib/prompts/`)
- `analysis-prompts.ts` - Prompt templates for AI analysis

## Core Concepts

### Job-Based Processing
- Tasks stored in-memory (no database persistence)
- Polling-based status updates via SWR
- Progress tracking: 0-100% in 7 steps
- Job cleanup after 24 hours (configurable)

### Two-Step AI Analysis

**Step 1: Generate Tag System**
- Analyzes all reviews together
- Generates 4-dimensional value model tag hierarchy
- Output: JSON structure with 3 levels of tags

**Step 2: Tag Individual Reviews**
- Applies generated tag system to each review
- Adds sentiment markers: [正面]/[负面] (positive/negative)
- Output: Tags with sentiment for each review

### Four-Dimensional Value Model

1. **人群与场景 (Crowd & Scenario)**: User characteristics, usage scenarios, purchase motivation, pain points
2. **功能价值 (Functional Value)**: Product advantages, disadvantages, suggestions, design
3. **保障价值 (Assurance Value)**: Logistics, after-sales, pre-sales service
4. **体验价值 (Experience Value)**: Recommendation willingness, brand impression, sensory experience, price perception

### Internationalization
- Full i18n support with next-intl
- URL-based language switching (`/zh/`, `/en/`)
- Auto-detect browser language preference
- AI analysis output matches current locale

## Environment Setup

Required environment variables (`.env.local`):

```env
# OpenAI-compatible API (Required)
OPENAI_API_KEY=your_api_key_here
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_MODEL=gpt-4o

# Browser mode (for frontend development only)
HEADLESS=false
```

### Supported API Providers

**OpenAI**:
```env
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_MODEL=gpt-4o
```

**智谱AI (GLM)**:
```env
OPENAI_BASE_URL=https://open.bigmodel.cn/api/paas/v4
OPENAI_MODEL=glm-4.6
```

**Other OpenAI-compatible APIs**: Configure `OPENAI_BASE_URL` accordingly.

## API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/comment-analysis/upload` | POST | Upload CSV and start analysis |
| `/api/jobs/[jobId]` | GET | Poll job status |
| `/api/comment-analysis/download/[filename]` | GET | Download processed CSV |
| `/api/comment-analysis/persona/[filename]` | GET | Download user persona (Markdown) |
| `/api/health` | GET | Health check |

## CSV Format Requirements

### Supported Column Names
System auto-identifies the review content column from:

**Chinese**: 评论内容, 评论, 内容
**English**: review_content, review, comment, content, body, text, message

### File Limitations
- Max file size: 16MB
- Max rows: 300
- Encodings: UTF-8, GBK (auto-detected)

### Output Columns Added
The system adds 16 new columns to the original CSV:
- 人群与场景: 使用场景, 购买动机, 未被满足的需求, 痛点问题
- 功能价值: 产品优点, 产品缺点, 用户期望建议, 设计与外观
- 保障价值: 物流配送, 售后服务, 售前服务
- 体验价值: 推荐意愿原因分析, 是否愿意推荐给他人, 品牌印象, 感官感受, 价格感知

## Important Notes

- **No persistence**: Jobs and files stored in-memory, server restarts lose all data
- **No Python required**: Pure Node.js/TypeScript implementation
- **API costs**: Each review requires one AI API call; costs scale with comment volume
- **Processing time**: 300 reviews may take 5-15 minutes depending on API speed
- **File storage**: Processed files stored in `uploads/processed/` directory

## Testing

### Test Data
Located in `test-data/`:
- `sample-reviews.csv` - Example CSV with 20 reviews
- `api-test.ts` - API testing script

### Run Tests
```bash
# Run API tests (requires server running)
npx tsx test-data/api-test.ts
```

## Common Development Tasks

### Add a new tag dimension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangdabiao/easy-amazon-voc](https://github.com/liangdabiao/easy-amazon-voc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
