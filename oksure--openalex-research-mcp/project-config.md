---
trigger: always_on
description: This file provides guidance for AI assistants (particularly Claude Code) when working with this codebase.
---

# Developer Guide for Claude Code

This file provides guidance for AI assistants (particularly Claude Code) when working with this codebase.

## Project Overview

An MCP (Model Context Protocol) server providing access to the OpenAlex API for literature review and research landscaping. Exposes 31 specialized tools through the MCP protocol, enabling AI assistants to search 240+ million scholarly works, analyze citations, track research trends, and map collaboration networks.

## Architecture

### Module Structure

1. **OpenAlex Client Layer** (`src/openalex-client.ts`)
   - Handles HTTP communication with OpenAlex API (api.openalex.org)
   - Manages rate limiting (10 req/s, 100K req/day) and 429 error handling with bounded retry
   - Implements "polite pool" access via email parameter or API key
   - Key methods: `getEntity()`, `searchEntities()`, `autocomplete()`, `getWorks()`, `getAuthors()`, etc.

2. **MCP Server Layer** (`src/index.ts`)
   - Implements stdio transport for MCP protocol
   - Defines 31 tools organized into 7 categories (see README for tool list)
   - Contains tool handler switch block

3. **Supporting Modules**
   - `src/presets.ts` — VENUE_PRESETS (9 journal/conference lists) and INSTITUTION_GROUPS (7 presets)
   - `src/formatters.ts` — Response formatting: `summarizeWork()`, `getFullWorkDetails()`, `reconstructAbstract()`, etc.
   - `src/filter.ts` — `buildFilter()` maps MCP tool params to OpenAlex filter format (exported for testing)
   - `src/config.ts` — Configuration constants and `debug()` helper (gated on `OPENALEX_DEBUG`)
   - `src/validation.ts` — Zod schemas for input validation

### Key Design Patterns

#### Response Optimization
- **List operations** (search, citations, etc.): Use `summarizeWork()` to return compact summaries
  - Authors limited to first 5 (with `authors_truncated` flag)
  - Only primary topic included
  - Abstracts truncated to 500 chars
  - Typical response: ~1.7 KB per work (down from ~10+ KB)

- **Single work retrieval** (`get_work`): Use `getFullWorkDetails()` to return complete information
  - ALL authors with positions (first, middle, last), institutions, ORCID IDs, corresponding author flags
  - Full abstract reconstructed from inverted index
  - All topics (not just primary)
  - Complete bibliographic data, funding, keywords
  - Use this when detailed author information is needed (e.g., identifying PIs, corresponding authors)

#### Parameter Translation
Tool parameters (e.g., `from_year`) are mapped to OpenAlex API filter format (e.g., `publication_year:>2019`) in the `buildFilter()` helper.

**CRITICAL**: OpenAlex uses `publication_year` filter, NOT `from_publication_date`
- Year ranges: `publication_year:2020-2023`
- From year: `publication_year:>2019` (note: use year-1 to be inclusive)
- To year: `publication_year:<2025`

#### Naming Conventions: `venue_*` vs `source_*`
- **`source_*` parameters** (`source_name`, `source_issn`, `source_id`) are used in generic search tools (`search_works`, `get_top_cited_works`, `find_review_articles`, etc.) where the venue filter is one of many optional filters.
- **`venue_*` parameters** (`venue_name`, `venue_issn`, `venue_id`) are used in venue-specific tools (`search_works_in_venue`, `check_venue_quality`) where the venue is the primary subject of the query.
- Both map to the same OpenAlex `primary_location.source.*` filters under the hood.

#### Other Patterns
- **Type Assertion**: Uses `const params = args as any` to handle MCP's unknown argument types
- **Citation Networks**: `get_citation_network` combines forward citations (via filter `cites:id`) and backward citations (from `referenced_works` field)
- **Collaborator Analysis**: `get_author_collaborators` paginates through author's works (up to 1000), then counts co-author occurrences across all authorships

## Development Commands

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run automated tests (REQUIRED before any release)
npm test

# Run comprehensive integration tests
npm run test:integration

# Run the server (after building)
npm start

# Development mode with auto-rebuild
npm run watch
```

## Testing

**CRITICAL: Always run tests before releasing or making changes.**

### Automated Testing

```bash
# Quick smoke tests (~15 seconds, required before every release)
npm test

# Full integration tests (~20 seconds, for major changes)
npm run test:integration
```

**Tests automatically run before `npm publish`** via `prepublishOnly` hook.

### Manual Testing with MCP Clients

**Claude Desktop:**
1. Build: `npm run build`
2. Add to config (`~/Library/Application Support/Claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "openalex": {
      "command": "node",
      "args": ["/absolute/path/to/openalex-research-mcp/build/index.js"],
      "env": {
        "OPENALEX_EMAIL": "your.email@example.com"
      }
    }
  }
}
```
3. Restart Claude Desktop
4. Test with: "Find the most influential papers on AI safety published since 2020"

**TypingMind or other MCP clients:** Same config format

## Publishing to npm

**ALWAYS follow this workflow:**

```bash
# 1. Make changes and build
npm run build

# 2. Run tests (REQUIRED - also runs automatically on publish)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oksure/openalex-research-mcp](https://github.com/oksure/openalex-research-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
