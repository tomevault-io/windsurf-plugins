---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**geodaddy**

Open source GEO (Generative Engine Optimization) analysis tool. Analyzes websites for AI-powered search engine optimization — helping content rank in ChatGPT, Perplexity, Google AI Overviews, and similar generative search engines. CLI-first, runs completely locally, outputs machine-readable JSON reports.

**Core Value:** Surface actionable GEO issues with specific fix recommendations — not just scores, but "here's what's wrong and exactly how to fix it."

### Constraints

- **Language**: Rust — single binary distribution, performance for crawling
- **Distribution**: Local CLI first, no cloud dependencies in v1
- **Output**: JSON-only for v1 (enables CI/CD integration, web UI can parse later)
- **Crawling**: Must handle localhost URLs for local development testing
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Executive Summary
- **Tokio** for async runtime (industry standard)
- **Reqwest** for HTTP client (11.5k stars, 294k projects)
- **Scraper** for HTML parsing (browser-grade via Servo)
- **Chromiumoxide** for headless browsing (async, comprehensive CDP coverage)
- **Clap** with derive macros for CLI (integrated structopt functionality)
## Core Framework & Runtime
### Async Runtime
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **tokio** | 1.49+ | Async I/O runtime | Industry standard for async Rust. 575M+ downloads. Mature ecosystem. Essential for concurrent HTTP requests. Built-in runtime metrics for detecting blocking operations. Use `full` feature for comprehensive capabilities. |
- De facto async runtime for Rust (2026 consensus)
- Excellent for I/O-bound crawling workloads
- Mature ecosystem with extensive middleware
- Built-in tracing integration
- `async-std`: Less ecosystem support in 2026
- `smol`: Lighter but smaller ecosystem
## HTTP Client & Web Crawling
### HTTP Client
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **reqwest** | 0.13+ | Async HTTP client | Ergonomic, batteries-included HTTP client built on hyper + tokio. 11.5k stars, 294k projects use it. Supports connection pooling, cookies, redirects, JSON serialization. Most mature option for web crawling. |
- Cookie storage (maintains session state)
- Connection pooling (built into async client)
- Customizable redirect handling
- HTTPS via rustls (default) or native-tls
- Timeout configuration (request-level and connection-level)
- Reuse client instances for connection pooling
- Configure timeouts: 30s request, 10s connection
- Enable gzip decompression
- Set custom User-Agent
- `hyper`: Lower-level, more boilerplate
- `surf`: Smaller ecosystem than reqwest
- `ureq`: Synchronous only
### Rate Limiting Middleware
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **reqwest-middleware** + **governor** | Latest | Rate limiting for polite crawling | Enables configurable throttling to avoid overwhelming target servers. Community standard pattern for crawlers. |
## HTML & Data Parsing
### HTML Parsing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **scraper** | 0.26+ | HTML parsing & CSS selectors | Browser-grade parsing via Servo's html5ever + selectors. 2.3k stars. Standards-compliant HTML5 parsing. CSS selector querying matches browser behavior. Latest release March 18, 2026. |
- HTML5-compliant parsing (same engine as Firefox)
- CSS selector support (standard selectors)
- DOM tree manipulation
- Text and attribute extraction
- Heading hierarchy analysis
- Schema.org markup extraction
- Content structure metrics
- Link discovery for crawling
- `html5ever`: Lower-level, requires more code
- `select.rs`: Less actively maintained
- `kuchiki`: Good but larger API surface
### XML/Sitemap Parsing
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **quick-xml** | 0.38+ | XML parsing for sitemaps | High-performance streaming XML parser. 10-50x faster than xml-rs. Event-based API for large files (e.g., Wikipedia dumps 1GB-100GB). Serde support for structured parsing. |
- Sitemap.xml parsing (sitemap-first strategy)
- Schema.org microdata extraction
- RSS/Atom feed discovery
- sitemapo adds complexity with auto-detection
- quick-xml provides more control for custom parsing
- Broader ecosystem usage (better maintained)
## Headless Browser Integration
### Chrome DevTools Protocol
| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| **chromiumoxide** | 0.9+ | Headless browser for JS rendering | Async/await support (tokio-based). Auto-generated type bindings from CDP spec (60k LOC generated). More comprehensive protocol coverage than rust-headless-chrome. Automatic Chromium download. Latest release Feb 25, 2026. |
- Full CDP protocol coverage (auto-generated from spec)
- Async API (integrates with tokio)
- Browser binary fetcher (downloads Chromium automatically)
- Type-safe command/event handling
- Sites with client-side rendering
- JavaScript-heavy SPAs
- Dynamic content loading
- Opt-in flag for geodaddy (adds complexity)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [borabiricik/geodaddy-cli](https://github.com/borabiricik/geodaddy-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
