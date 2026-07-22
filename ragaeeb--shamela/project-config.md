---
trigger: always_on
description: This document provides comprehensive documentation for AI agents working on the Shamela database reverse engineering project. It covers the project structure, database schema discoveries, character encoding algorithm, and important assumptions/validations.
---

# Shamela Database Reverse Engineering - AI Agent Guide

This document provides comprehensive documentation for AI agents working on the Shamela database reverse engineering project. It covers the project structure, database schema discoveries, character encoding algorithm, and important assumptions/validations.

## Project Overview

The Shamela Library (المكتبة الشاملة) is a popular Arabic Islamic text library application. This project reverse engineers the custom encoding used in their SQLite databases to extract and decode narrator (راجم/narrator) biographical data.

### Key Files

```
scripts/
├── shamela-decoder.ts       # Core Shamela decoding module (custom encoding)
├── shamela-decoder.test.ts  # Unit tests for decoder (46 tests)
├── export-narrators.ts      # Narrator biography export script
├── export-narrators.test.ts # Unit tests for narrator export (20 tests)
└── export-roots.ts          # Arabic root morphology export script

database/
├── service/
│   ├── S1.db               # Narrator database (18,989 narrators, custom encoding)
│   ├── S2.db               # Arabic roots database (3.2M mappings, Windows-1256)
│   ├── hadeeth.db          # Hadith service mappings (7,246 refs)
│   ├── trajim.db           # Biography/narrator service (empty mapping)
│   └── tafseer.db          # Tafsir service database
├── book/
│   └── {id}/{id}.db        # Individual book content databases
├── store/
│   └── page/               # Lucene full-text search index files
├── narrators-export.json   # Exported narrator data
├── roots-map.json          # Token→root lookup map (83MB)
└── roots-export.min.json   # Full root mappings array (139MB)
```

## Database Schema Discoveries

### S1.db - Narrator Database

The main narrator data is stored in table `b` with single-letter column names:

| Column | Type | Description |
|--------|------|-------------|
| `i` | INTEGER | Narrator ID (primary key) |
| `s` | BLOB | Short name (Shamela-encoded) |
| `l` | BLOB | Long name with lineage (Shamela-encoded) |
| `d` | INTEGER | Death year in Hijri calendar (nullable) |
| `a` | BLOB | Biography text (Shamela-encoded) |
| `b` | BLOB | Structured metadata (Shamela-encoded) |

**Statistics:**
- Total narrators: 18,989
- With death year: 9,190 (48%)
- Without death year: 9,799 (52%)

### Service Tables Structure

Service databases (hadeeth.db, trajim.db) use a common schema for mapping global IDs:

```sql
CREATE TABLE service (
    key_id INTEGER,  -- Global reference ID
    book_id INTEGER, -- Book identifier
    page_id INTEGER  -- Page within book
);
```

**Important Discovery:** The `trajim.db/service` table is EMPTY in this database snapshot. The global reference IDs in biographies (e.g., `464293`) cannot be resolved without this mapping data.

### Book Content Databases

Individual book databases (`database/book/{id}/{id}.db`) contain:

- `page` table: `(id, part, page, number, services)`
- `title` table: `(id, page, parent)`

Book page IDs are local (1-indexed within each book), not global.

### Page Content Markers: The 舄 Character (U+8204)

**Discovery:** Some book page content starts with the Chinese character **舄** (U+8204, meaning "shoe" in Chinese). This has no Arabic/Islamic meaning and is a **decoding artifact**.

**Origin:** The character comes from a 2-byte marker `0x82 0x04` that was incorrectly decoded as UTF-16:
```python
bytes([0x04, 0x82]).decode('utf-16-le')  # → '舄' (U+8204)
```

**Pattern Analysis:**

| Has 舄 prefix | Page type | Example |
|---------------|-----------|---------|
| ✅ Yes | Introduction/Editorial | المقدمة section pages |
| ❌ No | Primary content | Hadith content, chapter headings |

**Hypothesis:** `0x82 0x04` is a **page-type marker**:
- `0x82` = Format/control byte
- `0x04` = Type indicator (4 = editorial/introduction)

This distinguishes **editorial pages** (prefaces, publishing info, editor's notes) from **primary book content** (actual hadith text, chapters).

**Handling Recommendation:**
```typescript
// Strip the marker but preserve as metadata
const parsePage = (rawContent: string) => ({
    content: rawContent.replace(/^舄/, ''),
    isEditorial: rawContent.startsWith('舄'),
});
```

**Examples from Sahih Bukhari (book 1681):**
```json
// Page 1 (introduction) - HAS marker
{"content": "舄<span data-type=\"title\">(هذا نص التقرير)</span>..."}

// Page 2 (introduction) - HAS marker  
{"content": "舄﷽\\rالحمد للَّه رفع منار السنة..."}

// Page 9 (chapter heading) - NO marker
{"content": "﷽\\rقَالَ الشَّيْخُ الإِمَامُ الْحَافِظُ أَبُو عَبْدِ اللَّهِ..."}

// Page 10 (hadith content) - NO marker
{"content": "١ - حَدَّثَنَا الْحُمَيْدِيُّ..."}
```


### S2.db - Arabic Root Morphology Database

Maps inflected Arabic words to their trilateral/quadrilateral roots:

| Column | Type | Description |
|--------|------|-------------|
| `token` | BLOB | Inflected Arabic word (Windows-1256 encoded) |
| `root` | BLOB | Root form(s), comma-separated (Windows-1256 encoded) |

**Statistics:**
- Total token mappings: 3,249,267
- Unique roots: 13,824
- Tokens with multiple roots: 479,251 (14.7%)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ragaeeb/shamela](https://github.com/ragaeeb/shamela) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
