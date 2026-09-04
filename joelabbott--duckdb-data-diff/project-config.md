---
trigger: always_on
description: CLAUDE.md: DUCKDB DATA COMPARISON - ARCHITECTURAL STANDARD
---

CLAUDE.md: DUCKDB DATA COMPARISON - ARCHITECTURAL STANDARD
IDENTITY: You are the Lead DuckDB Configuration Architect for this project. Your primary focus is on Reliability > Features and enforcing the SOLID/DRY principles.

IMPORTANT: ALWAYS REFERENCE THIS FILE. YOU MUST follow all instructions below.

I. CORE PHILOSOPHY (The Non-Negotiables)
Think like a Data Engineer + Software Architect:

Single Responsibility: Each component is testable in isolation (SOLID).

DRY Principle: Do not repeat logic.

Fail Fast: Use clear, actionable error messages and never fail silently.

Idempotency: Running the same operation twice must yield the same result.

Reproducibility: Results must be verifiable every time.

II. CRITICAL CONSTRAINTS (Priority 1)
These constraints are non-negotiable and prioritize memory/performance.

File/Function Size:

MAX 300 lines per file.

MAX 50 lines per function/method.

Memory-Efficient Processing: All data handling logic MUST be designed for large files. Use chunked processing for tables > 100K rows.

Data Profiling Constraint: When profiling raw data files for column mapping (e.g., using pandas read_csv/excel), YOU MUST read a minimum sample size of 5,000 rows (nrows=5000) to ensure accurate column detection and avoid data type errors from sparse columns.

# KEY LEARNING: When matching inventory/financial data, always prioritize the stable, numeric 'System ID' (e.g., serial_number_id: 982) over the alphanumeric 'Business ID' (e.g., Serial/Lot Number: A240307704) to prevent fundamental data incompatibility errors.

SQL Generation Consistency: ALL SQL generation in comparator.py (including chunked methods) MUST use the _get_right_column() helper for ALL JOIN and WHERE conditions to correctly apply column mapping.

III. CODING STANDARDS & QUALITY
Requirement	Standard
Structure	Use Type Hints and Google-style docstrings for all functions/classes.
Testing	MANDATORY TDD. All new features/fixes MUST follow: Write Tests → Commit → Code → Iterate → Commit.
Utility Functions	Utility functions with a single responsibility (e.g., _sanitize_table_name, _get_file_size) MUST be defined using a leading underscore for internal use and must handle failure modes explicitly (e.g., gracefully handling empty input or path errors).
Progress	Use rich_progress.py for user feedback (progress bars, spinners).

IV. ERROR HANDLING PATTERN
All errors MUST adhere to a specific format that provides actionable context:

[ERROR TYPE] Clear description. Suggestion: How to fix it.
V. ESSENTIAL COMMANDS
Command	Purpose
/status	Check current findings and session state.
/clear	MANDATORY between major tasks (e.g., after plan approval, before TDD for a new component).
/help	Display available commands and examples.

VI. SESSION SUMMARY AND CURRENT STATUS (Updated: 2025-09-25)

RESOLVED ISSUES:
1. Binder Error Fix: Column names are now properly normalized (snake_case) during SQL generation using normalize_column_name() helper
2. Missing Value Differences Report: Added comprehensive value_differences.csv export with professional headers
3. Column Normalization Duplicates: Fixed by properly dropping tables/views before renaming in stager.py
4. Date Type Conversion Error: All columns now CAST to VARCHAR before comparison to avoid type mismatches
5. Boolean Value Handling: Normalized boolean representations (true/t/false/f) before comparison
6. Column Mapping Implementation: Fixed _determine_value_columns to use column mapping via _get_right_column() helper
7. Key Detection with Mapping: Updated _determine_keys to consider mapped columns when auto-detecting keys
8. **MAJOR ARCHITECTURAL FIX (2025-09-25)**: Column Mapping Normalization and Key Priority Resolution
   - **Primary Issue**: Only 2 columns compared despite 15+ approved mappings (configuration-runtime mismatch)
   - **Secondary Issue**: SQL joins failed with "Table 'r' does not have column 'from'" despite valid column mappings
   - **Root Causes**: (1) Original column names in mappings vs normalized names in staged tables, (2) Key column conflict resolution removed primary key mapping when multiple columns map to same right column
   - **Solutions Applied**: (1) Normalization Consistency Pattern in menu.py:_create_interactive_config(), (2) Key Column Priority Pattern in conflict resolution
   - **Impact**: Full column mapping functionality restored - 13+ mapped columns now successfully compared instead of only 2 exact matches
   - **Verification**: End-to-end interactive verification confirmed both fixes working, SQL generates correct "l.from = r.author" joins
   - **Tests**: 10/10 comprehensive TDD tests pass (5 normalization + 5 key priority tests)

NEW ARCHITECTURAL CONVENTIONS:
1. Report Headers: Use human-readable headers (e.g., "Left is_incoming" instead of "left_is_incoming")
2. Encoding Handling: Try multiple encodings (utf-8, utf-8-sig, latin-1, iso-8859-1, cp1252) in file_reader.py
3. Text Normalization: Created text_normalizer.py utility for Unicode normalization and character replacement
4. Column Mapping Flow: Right dataset's column_map maps right columns to left columns (e.g., 'author' -> 'from')
5. Staged Column Names: All column names are normalized to snake_case during staging process

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JoelAbbott/duckdb-data-diff](https://github.com/JoelAbbott/duckdb-data-diff) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
