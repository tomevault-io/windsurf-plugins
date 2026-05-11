---
trigger: always_on
description: Project context and development guidelines for AI-assisted development.
---

# BetterStructureSql - Claude Context

Project context and development guidelines for AI-assisted development.

## Project Purpose

Ruby gem that generates clean PostgreSQL schema dumps for Rails applications without pg_dump dependency. Replaces noisy structure.sql files with deterministic, maintainable output using pure Ruby database introspection.

## Core Problems Solved

**pg_dump issues**:
- Version-specific comments pollute git diffs
- Inconsistent output across PostgreSQL versions
- Cluster metadata creates noise
- External binary dependency
- Non-deterministic formatting

**Solution approach**:
- Pure Ruby implementation
- Query information_schema and pg_catalog directly
- Deterministic sorted output
- Clean SQL generation
- Optional schema versioning with retention management

## Component Architecture

### Core Components

**Configuration** - Centralized settings management with validation
- Output paths, search paths
- Feature toggles (extensions, functions, triggers, views)
- Schema versioning settings (enabled, retention limit)

**Introspection** - PostgreSQL metadata extraction
- Extensions from pg_extension
- Custom types and enums from pg_type
- Tables and columns from information_schema
- Indexes from pg_indexes
- Foreign keys from pg_constraint
- Views from pg_views
- Functions from pg_proc
- Triggers from pg_trigger

**Generators** - SQL statement creation (one per object type)
- ExtensionGenerator - CREATE EXTENSION statements
- TypeGenerator - CREATE TYPE for enums and domains
- TableGenerator - CREATE TABLE with columns and constraints
- IndexGenerator - CREATE INDEX with all variants
- ForeignKeyGenerator - ALTER TABLE ADD CONSTRAINT
- ViewGenerator - CREATE VIEW and MATERIALIZED VIEW
- FunctionGenerator - CREATE FUNCTION with plpgsql/sql
- TriggerGenerator - CREATE TRIGGER with timing and events

**Dumper** - Orchestration and file output
- Coordinates introspection
- Invokes generators in dependency order
- Formats output sections
- Writes structure.sql
- Triggers schema version storage

**Formatter** - Consistent SQL formatting
- Whitespace normalization
- Indentation management
- Keyword capitalization
- Section spacing

**SchemaVersions** - Version storage and retrieval
- Store schema snapshots in database
- MD5 hash-based deduplication (content_hash column)
- Skip storage when hash matches latest version
- Track PostgreSQL version and format type
- Manage retention with configurable limits
- Provide query interface for versions
- ZIP archive storage for multi-file schemas
- Extract and restore from stored versions

**DependencyResolver** - Object ordering (Not Currently Integrated)
- Class exists but not actively used in dumper
- Current implementation uses fixed section order (extensions → types → tables → views → functions → triggers)
- Fixed order works for most cases but may fail for complex inter-dependencies
- Future: Full dependency graph with topological sort for views/functions

**FileWriter** - Multi-file output management
- Detect output mode (file vs directory)
- Chunk sections into 500 LOC files with overflow handling
- Create numbered directories with load-order prefixes (1_extensions, 2_types, etc.)
- Write files incrementally for memory efficiency
- Generate numbered filenames (000001.sql, 000002.sql)

**ManifestGenerator** - Metadata for multi-file dumps
- Calculate statistics (total files, lines, breakdown by type)
- Generate load order respecting dependencies
- JSON format for tooling integration
- Parse and validate existing manifests

**ZipGenerator** - ZIP archive creation and extraction
- Uses rubyzip for in-memory ZIP operations
- Create from directory or file map
- Extract with path traversal protection
- Validation for ZIP bombs (file count, size limits)
- Stream large archives efficiently

**SchemaLoader** - Multi-format schema loading
- Auto-detect file vs directory mode
- Load multi-file using manifest order
- Stream large single files
- Support restoration from stored versions with temporary extraction

### Rails Integration

**Railtie** - Rails framework hooks
- Rake task registration
- Initializer loading
- Optional override of default schema dump

**Generator** - Installation scaffolding
- Create initializer configuration file
- Generate migration for schema_versions table
- Setup instructions

**Rake Tasks** - Command interface
- db:schema:dump_better - explicit dump
- db:schema:dump - replacement when configured
- db:schema:store - store version
- db:schema:versions - list versions
- db:schema:cleanup - manual retention cleanup

## Development Principles

### SOLID Principles

**Single Responsibility**
- Each generator handles exactly one PostgreSQL object type
- Introspection only queries metadata, never generates SQL
- Dumper orchestrates but delegates all generation
- Formatter handles presentation, not logic

**Open/Closed**
- New generators added without modifying existing code
- Configuration extensible via hash-like interface
- Hooks for before/after dump customization

**Liskov Substitution**
- All generators inherit from Base and implement generate(object)
- Interchangeable without affecting Dumper

**Interface Segregation**
- Small focused interfaces per component
- Generators only need generate method

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebyx07/better_structure_sql](https://github.com/sebyx07/better_structure_sql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
