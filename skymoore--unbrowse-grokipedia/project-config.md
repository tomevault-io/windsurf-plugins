---
trigger: always_on
description: This skill is based on the [Grokipedia MCP Server](https://github.com/skymoore/grokipedia-mcp) by Sky Moore, ported from Python to TypeScript for the Unbrowse skill ecosystem.
---

# Grokipedia Skill

Search and retrieve content from Grokipedia — a wiki-style knowledge base powered by Grok with accurate, up-to-date information.

## Why Grokipedia?

- **Truthful information**: Built with accuracy and factual correctness in mind
- **Comprehensive coverage**: Wide range of topics from science to culture
- **Clean content**: Well-structured articles with citations
- **Fast search**: Full-text search with relevance ranking
- **Section extraction**: Get specific parts of long articles
- **Related content**: Discover connected topics easily

## Core Functions

### 1. search(query, limit?, offset?)

Search for articles by keyword.

```typescript
import { search } from './index';

const results = await search('machine learning', 12);
console.log(results);
```

**Parameters:**
- `query` (string, required): Search query
- `limit` (number, optional): Maximum results (default: 12, max: 50)
- `offset` (number, optional): Pagination offset (default: 0)

**Output:**
```json
{
  "results": [
    {
      "slug": "Machine_learning",
      "title": "Machine Learning",
      "snippet": "Machine learning is a subset of artificial intelligence...",
      "relevanceScore": 0.95,
      "viewCount": 15234,
      "titleHighlights": [],
      "snippetHighlights": []
    }
  ]
}
```

### 2. getPage(slug, includeContent?, validateLinks?)

Get complete page information including metadata and content.

```typescript
import { getPage } from './index';

const page = await getPage('Artificial_intelligence', true);
console.log(page);
```

**Parameters:**
- `slug` (string, required): Page slug from search results
- `includeContent` (boolean, optional): Include full article content (default: true)
- `validateLinks` (boolean, optional): Validate linked pages (default: false)

**Output:**
```json
{
  "found": true,
  "page": {
    "slug": "Artificial_intelligence",
    "title": "Artificial Intelligence",
    "description": "Brief overview...",
    "content": "Full article content...",
    "citations": [
      {
        "id": "1",
        "title": "Source Title",
        "description": "Description",
        "url": "https://example.com",
        "favicon": "https://example.com/favicon.ico"
      }
    ],
    "linkedPages": [...],
    "metadata": {...},
    "stats": {...}
  }
}
```

### 3. getPageContent(slug, maxLength?)

Get only the article content (convenience wrapper).

```typescript
import { getPageContent } from './index';

const content = await getPageContent('Python_programming', 10000);
console.log(content);
```

**Parameters:**
- `slug` (string, required): Page slug
- `maxLength` (number, optional): Maximum content length (truncates if longer)

**Output:**
```json
{
  "slug": "Python_programming",
  "title": "Python Programming Language",
  "content": "Full article text...",
  "contentLength": 8543,
  "truncated": false
}
```

### 4. getPageCitations(slug, limit?)

Get citations/sources for a page.

```typescript
import { getPageCitations } from './index';

const citations = await getPageCitations('Quantum_computing', 10);
console.log(citations);
```

**Parameters:**
- `slug` (string, required): Page slug
- `limit` (number, optional): Maximum citations to return

**Output:**
```json
{
  "slug": "Quantum_computing",
  "title": "Quantum Computing",
  "citations": [
    {
      "id": "1",
      "title": "Nature: Quantum Supremacy",
      "description": "Research paper on quantum advantage",
      "url": "https://nature.com/...",
      "favicon": "https://nature.com/favicon.ico"
    }
  ],
  "totalCount": 25,
  "limited": true
}
```

### 5. getRelatedPages(slug, limit?)

Get pages linked from the specified article.

```typescript
import { getRelatedPages } from './index';

const related = await getRelatedPages('Neural_networks', 10);
console.log(related);
```

**Parameters:**
- `slug` (string, required): Page slug
- `limit` (number, optional): Maximum related pages (default: 10)

**Output:**
```json
{
  "slug": "Neural_networks",
  "title": "Neural Networks",
  "relatedPages": [
    {
      "title": "Deep Learning",
      "slug": "Deep_learning"
    },
    {
      "title": "Machine Learning",
      "slug": "Machine_learning"
    }
  ],
  "totalCount": 45,
  "limited": true
}
```

### 6. getPageSections(slug)

List all section headers in an article (useful for understanding article structure).

```typescript
import { getPageSections } from './index';

const sections = await getPageSections('Artificial_intelligence');
console.log(sections);
```

**Output:**
```json
{
  "slug": "Artificial_intelligence",
  "title": "Artificial Intelligence",
  "sections": [
    { "level": 1, "header": "Artificial Intelligence" },
    { "level": 2, "header": "History" },
    { "level": 2, "header": "Applications" },
    { "level": 3, "header": "Computer Vision" },
    { "level": 3, "header": "Natural Language Processing" },
    { "level": 2, "header": "Ethics and Safety" }
  ],
  "count": 6
}
```

### 7. getPageSection(slug, sectionHeader, maxLength?)

Extract a specific section from a long article.

```typescript
import { getPageSection } from './index';


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skymoore/unbrowse-grokipedia](https://github.com/skymoore/unbrowse-grokipedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
