---
trigger: always_on
description: AI Topic Radar is a daily AI topic discovery workflow for public-source monitoring, Chinese editorial topic scoring, and Markdown/JSON topic-pool output. A GitHub Actions cron job runs at 00:00 UTC (08:00 CST), gathers public AI signals, and produces digest files under `digests/YYYY-MM-DD/`.
---

# CLAUDE.md

## Project overview

AI Topic Radar is a daily AI topic discovery workflow for public-source monitoring, Chinese editorial topic scoring, and Markdown/JSON topic-pool output. A GitHub Actions cron job runs at 00:00 UTC (08:00 CST), gathers public AI signals, and produces digest files under `digests/YYYY-MM-DD/`.

## Commands

```bash
pnpm start          # run the full digest locally
pnpm test           # vitest (unit tests)
pnpm typecheck      # tsc --noEmit
pnpm lint           # ESLint
pnpm lint:fix       # ESLint --fix
pnpm format         # Prettier --write src
pnpm format:check   # Prettier --check src
```

Required env vars for local runs:

```bash
export GITHUB_TOKEN=ghp_xxxxx
export DIGEST_REPO=owner/repo   # omit to skip GitHub issue creation

# LLM provider (default: anthropic)
export LLM_PROVIDER=deepseek    # anthropic | openai | github-copilot | openrouter | deepseek
export DEEPSEEK_API_KEY=sk-xxxxx
export DEEPSEEK_MODEL=deepseek-chat

# Anthropic (default)
export ANTHROPIC_API_KEY=sk-ant-xxxxx
export ANTHROPIC_BASE_URL=https://api.kimi.com/coding/  # omit for Anthropic

# OpenAI
# export OPENAI_API_KEY=sk-xxxxx

# GitHub Copilot — uses GITHUB_TOKEN

# OpenRouter
# export OPENROUTER_API_KEY=sk-or-xxxxx
```

## Architecture

The pipeline runs in four sequential phases, each implemented as a named async function in `src/index.ts`:

1. **`fetchAllData`** — all network I/O in parallel: GitHub API (issues/PRs/releases), Claude Code Skills, Anthropic/OpenAI sitemaps, GitHub Trending HTML + Search API, Hacker News Algolia API, Product Hunt, arXiv, Hugging Face, Dev.to, and Lobsters.
2. **`generateSummaries`** — per-repo LLM calls, all in parallel, rate-limited to 5 concurrent requests by a queue in `src/report.ts`.
3. **Comparisons** — two LLM calls: cross-tool CLI comparison and OpenClaw cross-ecosystem comparison.
4. **Save phase** — `buildCliReportContent` / `buildOpenclawReportContent` (in `src/report-builders.ts`) build Markdown strings; `saveWebReport` / `saveTrendingReport` / `saveHnReport` (in `src/report-savers.ts`) call LLM + write file + create GitHub Issue; `buildTopicRadar` / `saveTopicRadar` (in `src/topic-radar.ts`) generate the Chinese editorial topic pool and JSON state.

## Source files

| File | Responsibility |
|------|---------------|
| `src/index.ts` | Orchestration: repo config, phase functions, `main()` |
| `src/i18n.ts` | Centralized bilingual strings: `Lang` type, report titles, issue labels, footer text, `REPORT_LABELS`, `NOTIFY_LABELS` |
| `src/github.ts` | GitHub API helpers: `fetchRecentItems`, `fetchRecentReleases`, `fetchSkillsData`, `createGitHubIssue`; shared `RepoFetch` type |
| `src/prompts.ts` | LLM prompt builders for repo reports: `buildCliPrompt`, `buildPeerPrompt`, `buildComparisonPrompt`, `buildPeersComparisonPrompt`, `buildSkillsPrompt` |
| `src/prompts-data.ts` | LLM prompt builders for data-source reports: `buildTrendingPrompt`, `buildWebReportPrompt`, `buildHnPrompt`, `buildWeeklyPrompt`, `buildMonthlyPrompt` |
| `src/report.ts` | `callLlm` (with concurrency limiter), `saveFile`, `autoGenFooter` (uses i18n), LLM token budget constants |
| `src/report-builders.ts` | `buildCliReportContent`, `buildOpenclawReportContent` — assemble final Markdown strings for CLI and OpenClaw reports |
| `src/report-savers.ts` | `saveWebReport`, `saveTrendingReport`, `saveHnReport` — LLM call + file save + optional GitHub issue |
| `src/date.ts` | Date and timing utilities: `toCstDateStr`, `toUtcStr`, `sleep` |
| `src/rollup.ts` | Weekly and monthly rollup report generator |
| `src/providers/types.ts` | `LlmProvider` interface, `ProviderName` type, `VALID_PROVIDER_NAMES` |
| `src/providers/openai-compatible.ts` | `OpenAICompatibleProvider` — shared base class for OpenAI-compatible providers |
| `src/providers/anthropic.ts` | `AnthropicProvider` — Anthropic SDK wrapper |
| `src/providers/openai.ts` | `OpenAIProvider` — extends `OpenAICompatibleProvider` |
| `src/providers/github-copilot.ts` | `GitHubCopilotProvider` — extends `OpenAICompatibleProvider` |
| `src/providers/openrouter.ts` | `OpenRouterProvider` — extends `OpenAICompatibleProvider` |
| `src/providers/index.ts` | `createProvider` factory + barrel re-exports |
| `src/web.ts` | Sitemap-based web content fetching; state persisted to `digests/web-state.json` |
| `src/trending.ts` | GitHub Trending HTML scraper + Search API topic queries |
| `src/hn.ts` | Hacker News top AI stories via Algolia HN Search API |
| `src/generate-manifest.ts` | Generates `manifest.json` (sidebar data for Web UI), `digests/search-index.json` (topic search data), and `feed.xml` (RSS 2.0 feed) |
| `src/topic-radar.ts` | Normalizes source data into an editorial topic pool, scores topics, and writes `ai-topic-radar.md` / `topic-pool.json` |
| `src/kr36.ts` | 36kr AI news articles fetched via RSS feed |
| `src/infoq-cn.ts` | InfoQ China AI articles fetched via internal API |
| `src/gitee.ts` | Gitee popular AI projects fetched via REST API v5 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Conradgui/AI-TREND-RADAR](https://github.com/Conradgui/AI-TREND-RADAR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
