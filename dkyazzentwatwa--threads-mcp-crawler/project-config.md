---
trigger: always_on
description: Internal documentation for developers working on `threads_scraper_mcp`.
---

# Threads Scraper MCP - Developer Guide

Internal documentation for developers working on `threads_scraper_mcp`.

## Architecture Overview

### Design Philosophy

1. **Robustness over cleverness**: Dual parsing (JSON + HTML) ensures resilience
2. **Centralized selectors**: All CSS selectors in one `ThreadsSelectors` class
3. **Graceful degradation**: Return partial data rather than failing completely
4. **Ethical scraping**: Built-in throttling and caching, no circumvention attempts
5. **Proven patterns**: Architecture mirrors successful `substack_mcp` implementation

### Tech Stack

- **Python 3.9+**: Modern type hints (`|` union, `Optional`, generics)
- **Pydantic v2**: Type-safe data validation
- **Playwright**: Headless Chrome browser automation (bypasses login wall)
- **BeautifulSoup + lxml**: HTML parsing (fallback)
- **VADER**: Sentiment analysis for social media
- **MCP SDK**: Model Context Protocol for AI integration
- **cachetools**: Thread-safe TTL caching

## Module Responsibilities

### `settings.py`
- Environment-based configuration via `@dataclass`
- `RuntimeSettings.from_env()` factory pattern
- Global `SETTINGS` instance
- **Env vars**: `THREADS_THROTTLE`, `THREADS_CACHE_TTL`, `THREADS_SESSION_COOKIE`, etc.

### `cache.py`
- **Copied verbatim from substack_mcp** (proven implementation)
- Thread-safe `TTLCache` with `RLock`
- `@cached` decorator for function memoization
- `clear_cache(prefix)` for selective invalidation
- 256-item limit, 10-minute TTL default

### `models.py`
- Pydantic v2 models for type safety
- **Core models**: `Profile`, `Thread`, `Reply`, `EngagementMetrics`
- **Analytics models**: `SentimentBreakdown`, `KeywordScore`, `ProfileAnalytics`
- All fields use `Optional` for graceful degradation
- `raw: Dict[str, Any]` field preserves original data for debugging

### `parsers.py`
- **CRITICAL FILE**: Most likely to need updates when Threads changes HTML
- `ThreadsSelectors` class: Centralized CSS selectors
- **Dual parsing system**:
  1. Primary: JSON from `__NEXT_DATA__` script tag
  2. Fallback: HTML selectors
- Helper functions: `_extract_next_data()`, `_parse_datetime()`, `_parse_engagement_from_json()`, etc.
- Main parsers: `parse_profile()`, `parse_thread()`, `parse_profile_threads()`, `parse_replies()`, `parse_search_results()`

### `client.py`
- `ThreadsPublicClient` class with **Playwright headless browser**
- **GraphQL interception**: Captures API responses from browser network
- **Throttling**: Timestamp-based, enforced in `_throttle()`
- **Lazy browser init**: Browser only starts on first request
- **@cached methods**: All fetch methods use decorator
- Error handling: Return `None`/empty list gracefully
- Methods: `get_profile()`, `get_profile_threads()`, `get_thread()`, `get_thread_replies()`, `search_threads()`
- **Key method**: `_fetch_with_graphql()` - navigates page and intercepts GraphQL responses

### `analysis.py`
- Analytics adapted for **short-form content**
- **VADER sentiment**: Compound score + pos/neg/neutral breakdown
- **Keyword extraction**: TF-based with stopword filtering
- **Hashtag extraction**: Separate regex capture, merged into keywords
- **Posting cadence**: Average days between posts
- **Engagement averages**: Mean likes/replies/reposts/quotes
- Returns `ProfileAnalytics` with raw data + computed metrics

### `mcp_server.py`
- MCP server setup with `Server("threads-scraper-mcp")`
- **6 tool handlers**: `get_profile`, `get_profile_threads`, `get_thread`, `get_thread_replies`, `search_threads`, `analyze_profile_threads`
- **Async/sync bridge**: `run_in_executor()` pattern
- **Error handling**: Try/catch with structured JSON errors
- **MCP bug workaround**: `create_text_result()` helper (serialization issue in MCP >1.10.0)

## Core Patterns

### 1. Dual Parsing System

**Why**: Threads uses React/NextJS with `__NEXT_DATA__` JSON blob for server-side rendering. This is more stable than HTML classes.

```python
def parse_profile(handle: str, html: str) -> Optional[Profile]:
    soup = BeautifulSoup(html, "lxml")
    next_data = _extract_next_data(soup)  # Try JSON first

    if next_data:
        user_data = next_data.get("props", {}).get("pageProps", {}).get("user")
        if user_data:
            return Profile(...)  # Structured data

    # Fallback to HTML
    name_elem = soup.find("h1")
    return Profile(display_name=name_elem.text, ...)
```

**Update strategy**:
1. When parsing fails, inspect `__NEXT_DATA__` JSON structure in browser DevTools
2. Update JSON path navigation in parsers
3. Only update HTML selectors if JSON structure completely changed

### 2. Centralized Selectors

**Why**: CSS classes change frequently. Centralizing makes updates fast.

```python
class ThreadsSelectors:
    """UPDATE HERE when HTML changes"""
    PROFILE_NAME = "h1[role='heading']"
    THREAD_TEXT = '[data-testid="thread-text"]'
    LIKE_COUNT = '[aria-label*="like"]'
```

**Update workflow**:
1. Identify broken selector (test failure or user report)
2. Inspect element in browser DevTools
3. Update `ThreadsSelectors` constant
4. No changes needed in parsing logic

### 3. Throttling Pattern

**Why**: Prevent rate limiting, respect ToS.

```python
def _throttle(self) -> None:
    delta = time.time() - self._last_request_ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dkyazzentwatwa/threads-mcp-crawler](https://github.com/dkyazzentwatwa/threads-mcp-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
