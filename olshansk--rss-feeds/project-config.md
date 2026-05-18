---
trigger: always_on
description: Instructions for Claude Code and contributors working on this repository.
---

# AGENTS.md <!-- omit in toc -->

Instructions for Claude Code and contributors working on this repository.

## Table of Contents <!-- omit in toc -->

- [Project Overview](#project-overview)
- [Commands](#commands)
- [Architecture](#architecture)
  - [Feed Generator Patterns](#feed-generator-patterns)
  - [When to Use Each Pattern](#when-to-use-each-pattern)
  - [Feed Link Setup (Important)](#feed-link-setup-important)
- [Adding a New Feed](#adding-a-new-feed)
  - [Step 1: Analyze the Target Blog](#step-1-analyze-the-target-blog)
  - [Step 2: Download HTML Sample](#step-2-download-html-sample)
  - [Step 3: Generate the Feed Script](#step-3-generate-the-feed-script)
  - [Step 4: Test Locally](#step-4-test-locally)
  - [Step 5: Register the Feed](#step-5-register-the-feed)
  - [Step 6: PR Checklist](#step-6-pr-checklist)
- [Deprecating a Feed](#deprecating-a-feed)
- [Troubleshooting](#troubleshooting)
- [GitHub Actions](#github-actions)

## Project Overview

RSS Feed Generator creates RSS feeds for blogs that don't provide them natively. Feed generators scrape blog pages and output `feed_*.xml` files to the `feeds/` directory. A GitHub Action runs hourly to regenerate and commit updated feeds.

## Commands

```bash
# Environment setup
make env_setup            # Install dependencies (uses uv sync)
make dev_setup            # Install dev dependencies + pre-commit hooks

# Generate feeds
make feeds_generate_all   # Run all feed generators
make feeds_<name>         # Run specific feed (e.g., feeds_ollama, feeds_anthropic_news)

# Development
make dev_lint             # Check code with ruff
make dev_lint_fix         # Auto-fix and format with ruff
make dev_format           # Alias for dev_lint_fix
make dev_test_feed        # Run test feed generator

# Run single generator directly
uv run feed_generators/ollama_blog.py

# CI/CD
make ci_trigger_feeds_workflow    # Trigger GitHub Action manually
make ci_run_feeds_workflow_local  # Test workflow locally with act
```

## Architecture

```
feed_generators/           # Python scripts that scrape blogs and generate RSS
  run_all_feeds.py         # Orchestrator that runs all generators
  utils.py                 # Shared utilities (setup_feed_links, get_project_root, etc.)
  <source>_blog.py         # Individual feed generators
feeds/                     # Output directory for feed_*.xml files
cache/                     # JSON cache for paginated/dynamic feeds
makefiles/                 # Modular Makefile includes (feeds.mk, env.mk, dev.mk, ci.mk)
```

### Feed Generator Patterns

Three patterns exist based on how the target site loads content:

#### 1. Simple Static (Default) <!-- omit in toc -->

For blogs where all content loads on first request.

**Examples**: `ollama_blog.py`, `paulgraham_blog.py`, `hamel_blog.py`

**Key functions**:
- `fetch_blog_content(url)` - HTTP request with User-Agent header
- `parse_blog_html(html)` - BeautifulSoup parsing for posts
- `generate_rss_feed(posts)` - Create feed using `feedgen`
- `save_rss_feed(fg, name)` - Write to `feeds/feed_{name}.xml`

**Cache**: Not needed (all posts fetched each run)

#### 2. Pagination + Caching <!-- omit in toc -->

For blogs with "Load More" or pagination that uses URL query params (`?page=2`).

**Examples**: `cursor_blog.py`, `dagster_blog.py`

**Key functions**:
- `load_cache()` / `save_cache(posts)` - JSON persistence in `cache/<source>_posts.json`
- `merge_posts(new, cached)` - Dedupe by URL, merge, sort by date
- `fetch_all_pages()` - Follow pagination until no next link

**Cache behavior**:
- **First run / `--full` flag**: Fetch all pages, populate cache
- **Incremental (default)**: Fetch page 1 only, merge with cache
- **Dedupe**: By URL, sorted by date descending

#### 3. Selenium + Click "Load More" <!-- omit in toc -->

For JS-heavy sites where content loads dynamically via JavaScript button clicks.

**Examples**: `anthropic_news_blog.py` (reference implementation), `anthropic_research_blog.py`, `openai_research_blog.py`, `xainews_blog.py`

**Key functions**:
- `setup_selenium_driver()` - Headless Chrome with `undetected-chromedriver`
- `fetch_news_content(max_clicks)` - Load page, click buttons, return final HTML
- `load_cache()` / `save_cache(articles)` - JSON persistence in `cache/<source>_posts.json`
- `merge_articles(new, cached)` - Dedupe by link, merge, sort by date

**Selenium specifics**:
- Uses `undetected-chromedriver` to avoid bot detection
- Clicks "See more"/"Load more" button repeatedly
- Waits for content to load between clicks
- `max_clicks` parameter controls depth (20 for full, 2-3 for incremental)

**Cache behavior** (see `anthropic_news_blog.py` for reference):
- **First run / `--full` flag**: Click up to 20 times, fetch all articles, populate cache
- **Incremental (default)**: Click 2-3 times (recent articles), merge with cache
- **Dedupe**: By URL, sorted by date descending

### When to Use Each Pattern

| Site Behavior | Pattern | Example | Cache? |
|--------------|---------|---------|--------|
| All posts on single page | Simple Static | `ollama_blog.py` | No |
| URL-based pagination (`?page=2`) | Pagination + Caching | `dagster_blog.py` | Yes |
| JS button loads more content | Selenium + Click | `anthropic_news_blog.py` | Yes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Olshansk/rss-feeds](https://github.com/Olshansk/rss-feeds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
