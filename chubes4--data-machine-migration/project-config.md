---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**Data Machine Migration** is a standalone WordPress utility plugin for migrating Data Machine ecosystem prefixes from `dm_` to `datamachine_` to meet WordPress repository requirements.

**Version**: 1.0.0
**Purpose**: One-time prefix migration across database, meta, options, and block content
**Lifecycle**: Install → Execute → Remove
**Requirements**: WordPress 5.0+, PHP 7.4+, Data Machine plugins installed

**Migration Status:**
- Phase 1: ✓ Complete (migration plugin ready)
- Phase 2: ✓ Complete - Core plugin code refactoring from `dm_` to `datamachine_` fully completed
- Phase 3: ✓ Complete - Database table names migrated to `wp_datamachine_*`
- Phase 4: ✅ Complete - Extension migrations deployed and verified

**Current Reality:** Core plugin and all extensions fully migrated to `datamachine_` prefixes. Migration plugin available for production deployment on existing installations.

## Core Architecture

### PSR-4 Autoloading
**Namespace**: `DataMachineMigration\`
**Class Pattern**: `Class_Name` → `class-class-name.php`
**Autoloader**: SPL autoloader in main plugin file

### Plugin Constants
```php
DATAMACHINE_MIGRATION_VERSION  // Plugin version
DATAMACHINE_MIGRATION_PATH     // Plugin directory path
DATAMACHINE_MIGRATION_URL      // Plugin URL for assets
```

### Class Structure

**Admin_Interface** (`class-admin-interface.php`)
- Admin menu registration under Tools → Data Machine Migration
- 6-step UI: Overview → Database → Meta → Options → Post Types → Blocks → Validation
- AJAX handler for migration execution
- JavaScript/CSS asset management
- Capability checks: `manage_options` required

**Database_Migrator** (`class-database-migrator.php`)
- Table rename operations: `wp_dm_*` → `wp_datamachine_*`
- Dependency-ordered execution (child tables first)
- Foreign key validation
- Row count verification
- Atomic RENAME TABLE operations

**Meta_Migrator** (`class-meta-migrator.php`)
- Post meta key migration with batch processing
- User meta key migration (single query)
- Recursive serialized data handling
- Batch size: 100 records per iteration
- Safety limit: 100,000 records maximum

**Options_Migrator** (`class-options-migrator.php`)
- WordPress options name and value migration
- Recursive serialized data processing
- String value replacement in nested structures
- Transient exclusion (skips `_transient%` options)

**Post_Type_Migrator** (`class-post-type-migrator.php`)
- Custom post type migration in wp_posts table
- Migrates `dm_events` → `datamachine_events`
- Batch processing: 100 posts per iteration
- Comprehensive validation and rollback support
- Safety limit: 100,000 records maximum

**Block_Migrator** (`class-block-migrator.php`)
- Gutenberg block reference updates
- Block mappings: `dm-events/*` → `datamachine-events/*`, `dm-recipes/*` → `datamachine-recipes/*`
- Post content and reusable blocks migration
- Batch processing: 50 posts per iteration
- Attribute key updates within block JSON

**Validation** (`class-validation.php`)
- Comprehensive post-migration validation
- Table existence and accessibility checks
- Meta key validation (ensures no old prefixes remain)
- Options validation
- Block content validation
- Performance checks with warnings

## Migration Execution Flow

### 6-Step Process

**Step 1: Database Tables**
```php
$migrator = new Database_Migrator();
$results = $migrator->migrate_tables();
```

**Execution Order** (dependency-aware):
1. `wp_dm_processed_items` → `wp_datamachine_processed_items` (child table)
2. `wp_dm_jobs` → `wp_datamachine_jobs`
3. `wp_dm_flows` → `wp_datamachine_flows`
4. `wp_dm_pipelines` → `wp_datamachine_pipelines` (parent table)

**Validation**: Row counts, foreign key relationships, SELECT query tests

**Step 2: Meta Keys**
```php
$migrator = new Meta_Migrator();
$results = $migrator->migrate_meta_keys();
```

**Processing**:
- Post meta: Batch processing (100 records/iteration) with serialized data handling
- User meta: Single SQL UPDATE with REPLACE function
- Recursive key replacement in nested arrays/objects

**Step 3: Options**
```php
$migrator = new Options_Migrator();
$results = $migrator->migrate_options();
```

**Processing**:
- Find options with `dm_` in name or value (excludes transients)
- Update option names: `dm_*` → `datamachine_*`
- Process serialized values recursively
- Delete old option names after successful migration

**Step 4: Post Types**
```php
$migrator = new Post_Type_Migrator();
$results = $migrator->migrate_post_types();
```

**Processing**:
- Update custom post types in wp_posts table
- `dm_events` → `datamachine_events`
- Batch processing: 100 posts per iteration
- Verification: Ensures zero posts remain with old type

**Step 5: Block Content**
```php
$migrator = new Block_Migrator();
$results = $migrator->migrate_block_content();
```

**Processing**:
- Update block namespaces in post content
- Migrate reusable blocks (wp_block post type)
- Update block attributes containing old prefixes
- Batch processing: 50 posts per iteration
- Note: Queries for datamachine_events (post types already migrated)

**Step 6: Validation**
```php

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chubes4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
