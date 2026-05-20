---
trigger: always_on
description: This document maintains continuity and understanding across conversation sessions about the DuckDB YAML extension project. It contains my current understanding, thoughts, questions, and ideas about the implementation. If we continue this project in a new conversation, reviewing this document will help me quickly understand the context and status of the project.
---

# CLAUDE.md - Project Notes for DuckDB YAML Extension

## Purpose of This Document

This document maintains continuity and understanding across conversation sessions about the DuckDB YAML extension project. It contains my current understanding, thoughts, questions, and ideas about the implementation. If we continue this project in a new conversation, reviewing this document will help me quickly understand the context and status of the project.

This is a high-level overview document. For detailed technical implementation notes and specific lessons learned, see CLAUDE_LESSONS.md.

## Project Overview

We are implementing a YAML extension for DuckDB, similar to the existing JSON extension, but using yaml-cpp instead of yyjson. The extension allows users to read YAML files into DuckDB tables and query the data using SQL.

## Current Implementation Status

- [x] Basic YAML file reading with read_yaml and read_yaml_objects functions
- [x] Multi-document YAML support
- [x] Top-level sequence handling (treating sequence items as rows)
- [x] File globbing and file list support
- [x] Error handling with partial recovery of valid documents
- [x] Comprehensive parameter handling with error checking
- [x] Direct file path support (SELECT * FROM 'file.yaml')
- [x] Test coverage for all implemented features
- [x] YAML logical type and conversion functions (to/from JSON)
- [x] Fix segfault in value_to_yaml function with debug mode implementation
- [x] JSON parity extraction functions (v1.5.0)
- [x] Arrow operator (`->>`) for string extraction
- [x] Function aliases for JSON compatibility
- [x] Extended multi-document modes (v1.6.0)
- [ ] Explicit column type specification via 'columns' parameter
- [ ] Comprehensive type detection (dates, timestamps, etc.)
- [ ] Support for YAML anchors and aliases
- [ ] Stream processing for large files

## Recent Changes and Findings

1. **YAML Type Implementation**:
   - Successfully implemented YAML as a type by extending VARCHAR with an alias
   - Used `LogicalType::SetAlias("yaml")` rather than creating a new type ID
   - Registered the type and appropriate cast functions
   - This approach provides a clean integration with DuckDB's type system

2. **Cast Functions and Type Conversions**:
   - Implemented proper cast functions between YAML, JSON, and VARCHAR
   - Added special treatment for multi-document YAML
   - Consistent handling of type conversions across all functions

3. **Display Format Improvements**:
   - Implemented YAML flow format (inline representation) for display purposes
   - Block format for storage and internal processing
   - Multi-document handling for both formats

4. **Code Structure Refactoring**:
   - Created a yaml_utils namespace for utility functions
   - Improved code organization with logical sections
   - Better error handling and resource management
   - Reduced code duplication through utility functions

5. **Fixed Segfault Issue**:
   - Added a debug mode implementation that prevents segfaults in value_to_yaml function
   - Created YAMLDebug class with safer alternative implementations
   - Added debug scalar functions for testing and diagnostic purposes
   - Implemented robust error handling with maximum recursion depth limits
   - Original segfault was related to stack overflow with deeply nested structures

6. **Testing Framework Challenges**:
   - Discovered SQLLogicTest framework constraints with multi-line strings
   - Addressed SQL string parsing issues by using flow-style YAML in tests
   - Found yaml-cpp's parser to be extremely resilient, handling malformed inputs
   - Adjusted test expectations for error handling given parser behavior
   - Updated error message expectations to match exact DuckDB error format

7. **v1.5.0 JSON Parity Functions**:
   - Added extraction functions to match DuckDB's JSON extension capabilities
   - **`->>` operator**: Arrow operator alias for `yaml_extract_string`
   - **`yaml_structure`**: Returns JSON schema of YAML document structure
   - **`yaml_contains`**: Recursive containment checking between YAML documents
   - **`yaml_merge_patch`**: RFC 7386 merge patch implementation
   - **`yaml_value`**: Extract scalar values only (NULL for arrays/objects)
   - **Function aliases**: `yaml_extract_path`, `yaml_extract_path_text`, `to_yaml`
   - Note: `->` operator cannot be implemented (DuckDB planner hardcodes it to `json_extract`)
   - Note: `yaml_transform` not implemented (requires binding-time type resolution); use `json_transform(yaml::JSON, structure)` instead

8. **v1.6.0 Extended Multi-Document Modes**:
   - Expanded `multi_document` parameter from boolean to support four modes:
   - **`true` / `'rows'`**: Each YAML document becomes a row (default, backward compatible)
   - **`false` / `'first'`**: Only the first document (backward compatible)
   - **`'frontmatter'`**: First document is metadata, rest are data rows
     - Metadata fields added as columns with `meta_` prefix

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teaguesterling/duckdb_yaml](https://github.com/teaguesterling/duckdb_yaml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
