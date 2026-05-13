---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DeepLX is a serverless translation service optimized for Cloudflare Workers. It provides a free alternative to translation APIs by proxying requests to DeepL and Google Translate services through intelligent load balancing, rate limiting, and caching mechanisms.

## Development Commands

### Running & Deployment
- `npm run dev` - Start local development server with Wrangler
- `npm run deploy` - Deploy to Cloudflare Workers production
- `npm run cf-typegen` - Generate TypeScript types for Cloudflare Workers bindings

### Testing
- `npm test` - Run all tests
- `npm run test:unit` - Run unit tests only (tests in tests/lib/)
- `npm run test:integration` - Run integration tests (tests/integration/)
- `npm run test:performance` - Run performance tests (tests/performance/)
- `npm run test:coverage` - Generate coverage report
- `npm run test:watch` - Run tests in watch mode
- `npm run test:debug` - Run tests with Node debugging enabled

### Code Quality
- `npm run lint` - Type-check with TypeScript (no emit, runs `tsc --noEmit`)

## Architecture

### Request Flow
1. Request arrives at Hono router (src/index.ts)
2. CORS preflight handling (if OPTIONS)
3. Security middleware validates input and extracts client IP
4. Rate limiting checks (two-level: client IP + proxy backend)
5. Cache lookup (two-level: in-memory + KV)
6. If cache miss:
   - Proxy manager selects available proxy endpoint
   - Circuit breaker checks if proxy is healthy
   - Retry logic with exponential backoff
   - Translation query to provider (DeepL or Google)
7. Cache successful translation
8. Return standardized response

### Core Components

#### Translation Providers
- **DeepL Query Engine** ([src/lib/query.ts](src/lib/query.ts)): Main translation logic for DeepL using JSONRPC API
- **Google Translate Service** ([src/lib/services/googleTranslate.ts](src/lib/services/googleTranslate.ts)): Google Translate integration

#### Support Systems
- **Two-Level Cache** ([src/lib/cache.ts](src/lib/cache.ts)): In-memory Map + Cloudflare KV with 1-hour TTL
- **Token Bucket Rate Limiter** ([src/lib/rateLimit.ts](src/lib/rateLimit.ts)): Dual rate limiting (per-client IP + per-proxy) with dynamic limits based on proxy count
- **Circuit Breaker** ([src/lib/circuitBreaker.ts](src/lib/circuitBreaker.ts)): Prevents cascade failures by temporarily blocking failing proxy endpoints (states: CLOSED/OPEN/HALF_OPEN)
- **Proxy Manager** ([src/lib/proxyManager.ts](src/lib/proxyManager.ts)): Random proxy selection with browser fingerprinting (User-Agent, Accept-Language rotation)
- **Retry Logic** ([src/lib/retryLogic.ts](src/lib/retryLogic.ts)): Exponential backoff retry mechanism (max 3 retries, 1s initial delay, 2x backoff factor)

#### Configuration
All configurable constants are centralized in [src/lib/config.ts](src/lib/config.ts):
- Rate limits: `RATE_LIMIT_CONFIG` (dynamic based on proxy count)
- Cache TTL: `CACHE_CONFIG`
- Retry settings: `DEFAULT_RETRY_CONFIG`
- Payload limits: `PAYLOAD_LIMITS`
- Request timeout: `REQUEST_TIMEOUT`

### API Endpoints

The application exposes three POST endpoints (defined in [src/index.ts](src/index.ts)):
- `/deepl` - DeepL translation (recommended)
- `/google` - Google Translate
- `/translate` - Legacy endpoint (uses DeepL for backward compatibility)
- `/debug` - Debug endpoint (only available when `DEBUG_MODE=true`)

All endpoints use the same `handleTranslation()` function with a provider parameter.

### Environment Configuration

Required environment variables in [wrangler.jsonc](wrangler.jsonc):
- `DEBUG_MODE` - Enable debug endpoint (default: "false")
- `PROXY_URLS` - Comma-separated list of XDPL proxy endpoints for DeepL

Required KV namespaces:
- `CACHE_KV` - Translation result cache
- `RATE_LIMIT_KV` - Rate limit token buckets

Required bindings:
- `ANALYTICS` - Cloudflare Analytics Engine dataset

### Scheduled Tasks

The worker includes a scheduled event handler that runs every 5 minutes (configured in `wrangler.jsonc` triggers):
- Clears in-memory cache to prevent memory leaks
- Called via `handleScheduled()` in [src/index.ts](src/index.ts)

## Key Implementation Details

### Rate Limiting Strategy
- **Client-level**: Token bucket with dynamic limit = `(proxy_count × 8 requests/sec) × 60 = tokens/minute`
- **Proxy-level**: 8 tokens/sec per proxy with 16 token burst capacity
- Uses two-level caching (in-memory 5s TTL + KV 1h TTL) for performance
- See `checkCombinedRateLimit()` in [src/lib/rateLimit.ts](src/lib/rateLimit.ts)

### Caching Strategy
- **In-memory cache**: JavaScript Map for fast lookups (cleared every 5 minutes by scheduled task)
- **KV cache**: Cloudflare KV for persistence (1-hour TTL)
- Cache key generation: hash of `text:source_lang:target_lang:provider`
- See [src/lib/cache.ts](src/lib/cache.ts)

### Circuit Breaker Pattern
- Opens after 5 consecutive failures
- 30-second recovery timeout
- Requires 3 consecutive successes to close
- One circuit breaker instance per proxy URL
- See [src/lib/circuitBreaker.ts](src/lib/circuitBreaker.ts)

### Security Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xixu-me/deeplx](https://github.com/xixu-me/deeplx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
