---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

**粵語辭叢 (Jyutjyu)** is an open Cantonese dictionary aggregation platform that unifies multiple dictionaries (classified, colloquialisms, etymology, etc.) into a single searchable interface. The platform supports intelligent search with traditional/simplified Chinese conversion, Jyutping romanization, and multi-dictionary queries.

### Key Technologies

- **Framework**: Nuxt 3 (Vue 3 + SSR)
- **UI**: Tailwind CSS
- **Data Storage**: Dual mode - Static JSON files or MongoDB Atlas
- **Chinese Conversion**: OpenCC.js (server and client)
- **Search**: MiniSearch (client) or MongoDB Atlas Search (server)
- **Deployment**: Vercel

## Development Commands

### Basic Development

```bash
npm run dev          # Start dev server on port 3000
npm run build        # Build for production
npm run generate     # Generate static site
npm run preview      # Preview production build
```

### Code Quality

```bash
npm run lint         # Run ESLint
npm run lint:fix     # Auto-fix ESLint issues
npm run typecheck    # Run TypeScript type checking
npm run test         # Run tests (Node.js built-in test runner)
```

Tests live in `tests/` as `.test.mjs` files. Run a single test with:

```bash
node --test tests/search-query-variants.test.mjs
```

### Data Processing

**CSV Validation** (before conversion):

```bash
npm run validate -- data/processed/your-file.csv
# Shortcut examples:
npm run validate:gzpc    # 实用广州话分类词典
npm run validate:hkcw    # 粤典
```

**CSV/JSONL to JSON Conversion**:

```bash
# Generic command
npm run build:data -- --dict <dict-id> --input <file>

# Dictionary-specific shortcuts:
npm run build:data:gzpc      # 分類詞典
npm run build:data:gzcl      # 俗語詞典
npm run build:data:gzwo      # 詞源
npm run build:data:gzd       # 方言詞典
npm run build:data:gzm       # 現代粵語
npm run build:data:gzdict    # 廣州話詞典
npm run build:data:hkcw      # 粵典
npm run build:data:qzjp      # 欽州粵拼
npm run build:data:kpd       # 開平方言
npm run build:data:tsed      # 台山話英文字典
npm run build:data:wiktionary  # 維基辭典
```

**Import to MongoDB**:

```bash
npm run db:import            # Add/update entries
npm run db:import:replace    # Replace entire dictionary
# Dictionary-specific:
npm run db:import:gzpc       # Import specific dictionary
```

## Architecture

### Dual Storage Mode

The application supports two storage modes, controlled by `NUXT_PUBLIC_USE_API` environment variable:

1. **Static JSON Mode** (`NUXT_PUBLIC_USE_API=false` or unset)
   - Dictionary data stored in `public/dictionaries/`
   - Client-side search using MiniSearch
   - Client-side OpenCC conversion
   - Suitable for development, demos, small-scale deployments
   - Zero database cost

2. **MongoDB API Mode** (`NUXT_PUBLIC_USE_API=true`)
   - Data stored in MongoDB Atlas
   - Server-side search via API endpoints
   - Server-side OpenCC conversion
   - Supports Atlas Search for full-text search
   - Suitable for production, large-scale deployments

3. **Auto Mode** (neither set): If `MONGODB_URI` is configured, defaults to API mode; otherwise uses static JSON.

**Implementation**: `composables/useSearch.ts` automatically selects the appropriate implementation based on runtime config.

### Core Data Flow

```
CSV Data (data/processed/)
  ↓ (scripts/csv-to-json.js with adapter)
JSON Files (public/dictionaries/)
  ↓ (optional: import-to-mongodb.js)
MongoDB Atlas
  ↓ (server/api/search.ts)
Client Search (composables/useSearch.ts)
```

### Dictionary Data Schema

Core type: `DictionaryEntry` (defined in `types/dictionary.ts`)

Key fields:

- `id`: Unique identifier
- `source_book`: Dictionary source
- `dialect`: Dialect information
- `headword`: `{ display, normalized, search }` - handles variant spellings
- `phonetic`: `{ jyutping[], original }` - Jyutping arrays for polyphonic characters
- `senses`: Array of definitions with examples
- `keywords`: Search optimization array (generated during build)
- `meta`: Dictionary-specific metadata (category, etymology, references, etc.)

### Search Logic

**Normal Mode Priority**:

1. Exact headword match (priority 100)
2. Prefix match (90)
3. Contains match (80)
4. Exact Jyutping match (70)
5. Jyutping contains match (60)
6. Keyword match (50)

**Reverse Mode** (search by definition):

- Exact definition match (100)
- Definition contains match (80)

**Secondary sorting**: Entry length, definition detail, dictionary quality weight

Implementation:

- Client: `composables/useDictionary.ts` - `searchBasic()` method
- Server: `server/api/search.ts` - `fallbackSearch()` and `atlasSearch()` functions

### Dictionary Adapters

Each dictionary has a custom adapter in `scripts/adapters/` that transforms CSV rows into the standard `DictionaryEntry` format. Adapters handle:

- Dictionary-specific metadata mapping
- Jyutping normalization
- Definition and example parsing
- Reference link generation

When adding a new dictionary, create a new adapter following the pattern in existing adapters.

### Caching Strategy

**Client-side** (`composables/useSearch.ts`):

- Search results cached for 30 minutes (max 50 queries)
- Dictionary data cached globally

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyutjyucom/Jyutjyu](https://github.com/jyutjyucom/Jyutjyu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
