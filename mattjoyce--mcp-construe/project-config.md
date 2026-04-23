---
trigger: always_on
description: This FastMCP server provides tools to extract and concatenate personal context from an Obsidian vault. The primary use case is loading curated personal knowledge into an LLM conversation by leveraging Obsidian's frontmatter tagging system.
---

# FastMCP Obsidian Context Server - Development Brief

## Overview
This FastMCP server provides tools to extract and concatenate personal context from an Obsidian vault. The primary use case is loading curated personal knowledge into an LLM conversation by leveraging Obsidian's frontmatter tagging system.

## Core Functionality

### Purpose
- Load personal context from Obsidian notes into LLM conversations
- Use Obsidian frontmatter properties and tags as filtering criteria
- Provide concatenated, chronologically ordered content for comprehensive context

### Architecture
The server consists of several main tools:
1. **fetch_context()** - Loads context based on context type (e.g., 'personal', 'work')
2. **fetch_matching_files()** - Flexible tool for runtime filtering by properties/tags
3. **fetch_frontmatter_index()** - Lightweight metadata browsing without full content
4. **fetch_specific_file()** - Targeted retrieval of specific files by path
5. **search_vault_content()** - Content-based search (hardcore mode) with frontmatter results

## Configuration System

### config.yaml Structure
```yaml
vault_path: "/path/to/obsidian/vault"
default_context:
  properties:
    context: "personal"
  tags: []
```

### Configuration Loading
- config.yaml must be located in the same directory as the MCP server script
- Server reads configuration on startup
- Vault path supports user home directory expansion (~)

## File Processing Pipeline

### 1. File Discovery
- Recursively scan vault directory for `.md` files using pathlib
- Filter out non-markdown files automatically

### 2. Frontmatter Parsing
- Extract YAML frontmatter between first and second `---` at line start
- Parse properties and tags from frontmatter
- Handle malformed YAML gracefully (skip file or log warning)

### 3. Filtering Logic
**Properties Matching:**
- AND logic: All specified properties must match exactly
- Case-sensitive property names and values

**Tags Matching:**
- Default OR logic: File matches if it contains any specified tag
- Optional AND logic: `match_all_tags=True` requires all specified tags
- Tags expected as YAML list in frontmatter: `tags: [personal, finance]`

**Combined Filtering:**
- Properties AND tags criteria must both be satisfied
- Empty criteria arrays are ignored (no filtering applied)

### 4. Sorting
- Sort matched files by file modification time (oldest first)
- Use pathlib.Path.stat().st_mtime for consistent cross-platform behavior

### 5. Content Concatenation
**Format:**
```
================================================================================
/absolute/path/to/file.md
================================================================================
[full file content including frontmatter]

================================================================================
/absolute/path/to/next/file.md
================================================================================
[full file content including frontmatter]
```

## Tool Specifications

### fetch_context()
- **Parameters:**
  - `context_type: str` - Context type to match (e.g., 'personal', 'work')
  - `chunk_index: int = 0` - Which chunk to retrieve
  - `max_chars: int = 95000` - Maximum characters per chunk
- **Returns:** String (concatenated content)
- **Behavior:** Uses context type to filter by context property from config.yaml
- **Error Handling:** Returns error message if config missing or vault inaccessible

### fetch_matching_files()
- **Parameters:**
  - `properties: dict` - Key-value pairs to match in frontmatter
  - `tags: list[str]` - Tags to search for
  - `match_all_tags: bool = False` - Whether to require all tags (AND) vs any tags (OR)
  - `chunk_index: int = 0` - Which chunk to retrieve
  - `max_chars: int = 95000` - Maximum characters per chunk
- **Returns:** String (concatenated content)
- **Behavior:** Runtime filtering with specified criteria, with chunking support
- **Error Handling:** Returns error message for invalid parameters or processing errors

### fetch_frontmatter_index()
- **Parameters:**
  - `properties: dict` - Key-value pairs to match in frontmatter
  - `tags: list[str]` - Tags to search for
  - `match_all_tags: bool = False` - Whether to require all tags (AND) vs any tags (OR)
- **Returns:** String (formatted table of file metadata)
- **Behavior:** Returns lightweight frontmatter index without full content - ideal for browsing
- **Error Handling:** Returns error message for invalid parameters or processing errors

### fetch_specific_file()
- **Parameters:**
  - `file_path: str` - Absolute or relative path to the file
- **Returns:** String (complete file content with formatting header)
- **Behavior:** Retrieves full content of a specific file by path - use after browsing index
- **Error Handling:** Returns error message for invalid paths or inaccessible files

### search_vault_content() (Hardcore Mode)
- **Parameters:**
  - `search_pattern: str` - Text or regex pattern to search for in file content
  - `case_sensitive: bool = False` - Whether to perform case-sensitive search

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mattjoyce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
