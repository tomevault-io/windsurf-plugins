---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Type: Python Library

**feedsearch-crawler** is a **library package** designed to be consumed by other Python projects. It is NOT an end-user application.

### Key Implications

1. **Stability is Critical**: Breaking changes affect downstream users
2. **API Surface Matters**: All public APIs must be well-documented and tested
3. **Dependencies Must Be Minimal**: Keep dependency tree small to avoid conflicts
4. **Backward Compatibility**: Use semantic versioning and deprecation warnings
5. **Build Size Matters**: Distributed via PyPI, keep package lean

### Target Audience

- Python developers integrating feed discovery into their applications
- Web scraping and content aggregation tools
- RSS/Atom feed readers and aggregators
- API services that discover feeds (like feedsearch.dev)

### Public API Surface

The library exposes these public APIs in `__init__.py`:
- `search(url, ...)` - Synchronous feed search
- `search_async(url, ...)` - Async feed search
- `output_opml(feeds)` - Convert feeds to OPML format
- `FeedInfo` - Feed metadata class with `serialize()` method
- `FeedsearchSpider` - Main spider class (advanced usage)

**Changes to these must maintain backward compatibility.**

## Commands

### Development with uv

This project uses [uv](https://docs.astral.sh/uv/) for package management and development:

- `uv sync` - Install dependencies and sync the environment
- `uv run ruff check` - Run linting checks
- `uv run ruff format` - Format code
- `uv run pytest` - Run all tests
- `uv run pytest --durations=20` - Show 20 slowest tests
- `uv run pytest tests/crawler/test_request.py` - Run a specific test file
- `uv run pytest tests/crawler/test_request.py::TestRequest::test_method_name` - Run a specific test

### Running the Application

- `uv run main.py` - Run with default URLs from file
- `uv run main.py https://example.com` - Crawl single URL
- `uv run main.py example.com` - Crawl single URL with domain only
- `uv run main.py https://site1.com https://site2.com` - Crawl multiple URLs
- `uv run main.py --urls "https://site1.com,https://site2.com"` - Use comma-separated format
- `uv run main.py --help` - Get help

### Package Installation

The library is available on PyPI: `pip install feedsearch-crawler`

### Publishing to PyPI

**Prerequisites:**
- Maintainer access to feedsearch-crawler on PyPI
- PyPI API token configured

**Publishing workflow:**

```bash
# 1. Update version in pyproject.toml
# Follow semantic versioning: MAJOR.MINOR.PATCH

# 2. Update CHANGELOG.md with release notes

# 3. Run full test suite
uv run pytest

# 4. Build the package
uv build

# 5. Check build artifacts
ls dist/
# Should see: feedsearch_crawler-X.Y.Z-py3-none-any.whl
#             feedsearch_crawler-X.Y.Z.tar.gz

# 6. Upload to TestPyPI (optional but recommended)
uv publish --token $TEST_PYPI_TOKEN --publish-url https://test.pypi.org/legacy/

# 7. Test installation from TestPyPI
pip install --index-url https://test.pypi.org/simple/ feedsearch-crawler

# 8. Upload to PyPI
uv publish --token $PYPI_TOKEN

# 9. Create git tag
git tag v1.0.3
git push origin v1.0.3

# 10. Create GitHub release with CHANGELOG notes
```

**Semantic versioning guidelines:**
- **MAJOR** (1.0.0 → 2.0.0): Breaking API changes
- **MINOR** (1.0.0 → 1.1.0): New features, backward compatible
- **PATCH** (1.0.0 → 1.0.1): Bug fixes, backward compatible

**Before publishing checklist:**
- [ ] All tests pass
- [ ] CHANGELOG.md updated
- [ ] Version bumped in pyproject.toml
- [ ] README.md examples still work
- [ ] No debug code or print statements
- [ ] Dependencies properly specified

## Architecture Overview

### Core Components

**feedsearch-crawler** is an asyncio-based RSS/Atom/JSON feed discovery crawler built on two main architectural layers:

#### 1. Crawler Framework (`src/feedsearch_crawler/crawler/`)

A generic asynchronous web crawler framework providing:

- **Crawler**: Base class for web crawlers with request/response handling, middleware support, and concurrent processing
- **Request/Response**: HTTP request and response objects with proper typing
- **Downloader**: HTTP client wrapper using aiohttp with connection pooling and error handling
- **Middleware**: Pipeline-based request/response processing (robots.txt, throttling, retry logic, cookies, content-type handling, monitoring)
- **DuplicateFilter**: URL deduplication to avoid crawling the same resource multiple times
- **Item/ItemParser**: Structured data extraction and processing

#### 2. Feed Spider (`src/feedsearch_crawler/feed_spider/`)

Specialized crawler for feed discovery built on the crawler framework:

- **FeedsearchSpider**: Main spider class that orchestrates feed discovery
- **FeedInfo**: Data class representing discovered feeds with metadata (title, description, score, etc.)
- **FeedInfoParser**: Parses and validates feed content (RSS, Atom, JSON feeds)
- **SiteMeta/SiteMetaParser**: Extracts website metadata and favicon information
- **LinkFilter**: Filters and prioritizes potential feed URLs
- **Favicon**: Handles favicon discovery and data URI conversion

### Key Design Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DBeath/feedsearch-crawler](https://github.com/DBeath/feedsearch-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
