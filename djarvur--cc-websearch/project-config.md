---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->

## Project

**cc-websearch**

A Claude Code plugin providing two skills that replace the built-in WebSearch and WebFetch tools. Distributed as a standard Claude Code plugin installable via the plugin command. Two Node CLI scripts (`websearch`, `webfetch`) called via `node` from skill definitions, producing output identical to Claude Code's built-in tools.

**Core Value:** Exact drop-in replacement for Claude Code's WebSearch and WebFetch — same interface, same output format, no behavior changes for the user.

### Constraints

- **Runtime**: TypeScript/Node — scripts run via `node`
- **Distribution**: Standard Claude Code plugin — installable via plugin command
- **Output format**: Must match Claude Code's WebSearch and WebFetch output byte-for-byte
- **Perplexity API**: Chat Completions endpoint
- **DDG API**: DuckDuckGo Lite HTML scraping
- **Config**: `~/.config/websearch/config.json` or environment variables
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->

## Technology Stack

## Recommended Stack

### Core Technologies

| Technology                     | Version | Purpose                       | Why Recommended                                                                                                                                                                                        |
| ------------------------------ | ------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| TypeScript                     | 5.x     | Language                      | Claude Code plugins run via `node`; TypeScript provides type safety for CLI input parsing, API response handling, and config validation. The ecosystem strongly favors TS.                             |
| Node.js                        | 20 LTS+ | Runtime                       | Required by Claude Code plugin system (`node` command in skill definitions). v20 LTS is the current long-term support line.                                                                            |
| Commander.js                   | 13.x    | CLI argument parsing          | De facto standard for Node CLI apps. Supports hybrid input: flags for simple queries plus `parseAsync()` for programmatic use. TypeScript-native, zero dependencies, well-maintained.                  |
| `@perplexity-ai/perplexity_ai` | 2.x     | Perplexity API client         | Official Perplexity TypeScript SDK. Supports Chat Completions (sonar models) with built-in retries, timeouts, error handling, and streaming. OpenAI-compatible response format with `citations` array. |
| `duck-duck-scrape`             | 2.2.x   | DuckDuckGo fallback           | Node.js library for scraping DDG search results without API keys. Supports text, image, video, news search. The only maintained DDG scraping library in the ecosystem.                                 |
| `turndown`                     | 7.2.x   | HTML to Markdown conversion   | Standard HTML-to-Markdown converter for JavaScript. Used by Firefox, Joplin, and countless tools. Extensible rule system, plugin support for GFM tables.                                               |
| `@mozilla/readability`         | 0.6.x   | Content extraction            | Firefox Reader View engine. Extracts article content from HTML, stripping navigation, ads, sidebars. The proven solution for clean content extraction.                                                 |
| `jsdom`                        | 25.x    | DOM parsing for Readability   | Required by `@mozilla/readability` (needs a `document` object). jsdom provides a W3C-compliant DOM in Node.js. The standard pairing with Readability.                                                  |
| `cheerio`                      | 1.x     | HTML parsing for DDG scraping | Lightweight jQuery-style HTML parser. Used to parse DuckDuckGo Lite HTML search results. Far lighter than jsdom for simple scraping -- no full DOM needed.                                             |

### Supporting Libraries

| Library               | Version | Purpose                        | When to Use                                                                                                                        |
| --------------------- | ------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `zod`                 | 3.x     | Schema validation              | Validate JSON stdin input matching Claude Code's WebSearch/WebFetch tool schemas. Runtime type checking with TypeScript inference. |
| `turndown-plugin-gfm` | 1.0.x   | GFM table support for Turndown | Always -- web pages contain tables. Enables GitHub Flavored Markdown table conversion.                                             |
| `p-limit`             | 6.x     | Concurrency control            | If batching multiple DDG scrape requests. Prevents rate-limiting from aggressive parallel requests.                                |

### Development Tools


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Djarvur/cc-websearch](https://github.com/Djarvur/cc-websearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
