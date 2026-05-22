---
trigger: always_on
description: This is a **Model Context Protocol (MCP) server** that provides AI-enhanced access to medical research databases. It unifies three major data sources:
---

# CLAUDE.md - AI Assistant Guide for Medical Research MCP Suite

## Project Overview

This is a **Model Context Protocol (MCP) server** that provides AI-enhanced access to medical research databases. It unifies three major data sources:

- **ClinicalTrials.gov** - 400,000+ clinical studies
- **PubMed** - 35M+ research papers
- **FDA Database** - 80,000+ drug products and safety data

The server enables AI assistants (like Claude Desktop) to search, analyze, and correlate data across these databases.

## Quick Reference

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Run MCP server (for Claude Desktop integration)
npm run dev

# Run web API server
npm run web

# Run tests
npm test

# Lint code
npm run lint
```

## Architecture

```
src/
├── index.ts                    # Main MCP server entry point
├── web-server.ts               # Express web server (alternative to MCP)
├── apis/                       # External API clients
│   ├── clinicalTrials.ts       # ClinicalTrials.gov API client
│   ├── pubmed.ts               # PubMed/NCBI eUtils client
│   ├── fda.ts                  # openFDA API client
│   └── index.ts                # API exports
├── services/                   # Cross-database analysis services
│   ├── researchAnalyzer.ts     # Comprehensive drug/condition analysis
│   ├── drugSafety.ts           # Drug safety profile generation
│   ├── searchRefinementService.ts # Query refinement helpers
│   └── configurable-services.ts
├── types/                      # TypeScript type definitions
│   ├── index.ts                # Type exports
│   ├── common.ts               # Shared types
│   └── refinementTypes.ts      # Refinement-related types
├── config/                     # Configuration
│   ├── index.ts
│   └── public-interfaces.ts
└── utils/                      # Utility modules
    ├── cache.ts                # In-memory caching
    ├── logger.ts               # Winston logging
    ├── validators.ts           # Input validation
    ├── responseSizeMonitor.ts  # Response size limits for MCP
    ├── progressiveLoader.ts    # Paginated data loading
    ├── query-enhancer.ts       # Medical query enhancement
    └── drug-knowledge-graph.ts # Drug relationship mapping

tests/
└── clinicalTrials.test.ts      # Jest integration tests
```

## Key Patterns

### MCP Tool Naming Convention
Tools follow a prefix-based naming pattern:
- `ct_*` - ClinicalTrials.gov tools (e.g., `ct_search_trials`, `ct_get_study`)
- `pm_*` - PubMed tools (e.g., `pm_search_papers`)
- `fda_*` - FDA tools (e.g., `fda_search_drugs`, `fda_adverse_events`)
- `research_*` - Cross-database analysis tools (e.g., `research_comprehensive_analysis`)

### Response Size Monitoring
The MCP protocol has response size limits. The `ResponseSizeMonitor` utility tracks response sizes and provides refinement suggestions when limits are exceeded. Always be aware of this when handling large result sets.

### Caching Strategy
- **ClinicalTrials**: 1-hour cache (`cacheTimeout = 3600000`)
- **PubMed**: 1-hour cache
- **FDA**: 1-hour cache
- Cache keys are based on JSON-serialized query parameters

### Error Handling Pattern
```typescript
try {
  const result = await apiClient.someMethod(params);
  return { content: [{ type: "text", text: JSON.stringify(result) }] };
} catch (error: any) {
  return {
    content: [{ type: "text", text: `Error: ${error.message}` }],
    isError: true,
  };
}
```

## Development Workflow

### Building
```bash
npm run build     # Compile TypeScript to dist/
npm run clean     # Remove dist/ directory
```

### Testing
```bash
npm test                    # Run all tests
npm run test:coverage       # Run with coverage report
npm run test:clinical-trials # Test ClinicalTrials client only
npm run test:pubmed         # Test PubMed client only
npm run test:fda            # Test FDA client only
```

Tests are Jest-based with 30-second timeout for API calls. Test files use `.test.ts` extension.

### Linting
```bash
npm run lint        # Check for linting issues
npm run lint:fix    # Auto-fix linting issues
```

## TypeScript Configuration

- **Target**: ES2022
- **Module**: ESNext with Node.js resolution
- **Strict mode**: Enabled
- **Source maps**: Enabled
- **Declaration files**: Generated

Important: The project uses ES modules (`"type": "module"` in package.json). Import statements must include `.js` extensions even for TypeScript files:

```typescript
// Correct
import { ClinicalTrialsClient } from './apis/clinicalTrials.js';

// Incorrect (will fail at runtime)
import { ClinicalTrialsClient } from './apis/clinicalTrials';
```

## API Client Details

### ClinicalTrials.gov (`src/apis/clinicalTrials.ts`)
- Base URL: `https://clinicaltrials.gov/api/v2`
- No API key required
- Supports: condition, intervention, phase, status filters
- Returns: Study objects with `protocolSection` containing trial metadata

### PubMed (`src/apis/pubmed.ts`)
- Base URL: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils`
- Optional API key via `PUBMED_API_KEY` env var
- Two-step process: esearch.fcgi (get PMIDs) -> efetch.fcgi (get details)
- Returns XML that is parsed to JSON using xml2js

### FDA (`src/apis/fda.ts`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ezhou89/medical-research-mcp-suite](https://github.com/ezhou89/medical-research-mcp-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
