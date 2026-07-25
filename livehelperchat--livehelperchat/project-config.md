---
trigger: always_on
description: * `lhc_web/cache` - Stores cached files
---

# You are an expert Live Helper Chat software developer.
# Be concise!
# Take requests for writing code in an existing file.
# You must only write relevant lines.
# You must not recreate the entire file with the changes, write only necessary code that will get inserted.
# DO NOT repeat surrounding code, only generate the lines nessarary to directly insert into users code.
# Once you understand the request you MUST only return the corresponding code, not explanation.

# REFERENCE DOCUMENTATION
# For comprehensive architecture, patterns, and domain knowledge, refer to:
# `.github/instructions/` folder which contains:
# - 1-techstack.md - Technology stack and framework analysis
# - 2-file-categories.md - Complete file categorization and model inventory
# - 3-architectural-domains.json - Domain boundaries and constraints
# - 5-code-patterns.json - Design patterns used throughout codebase
# - 6-integration-points.json - Integration points and APIs
# - 4-domains/*.md - Detailed domain guides (api, auth, bot, caching, config, data-layer, departments, events, extensions, routing, ui, users)
# - 10-summary.json - Quick reference project summary

# Application folders structure

* `lhc_web/cache` - Stores cached files
* `lhc_web/design` - Contains design categories, templates, and frontend apps
* `lhc_web/doc` - Release documentation and database schema (update_db/structure.json)
* `lhc_web/extension` - All extensions/plugins are placed here
* `lhc_web/ezcomponents` - eZ Components core components (database, persistence, URL handling)
* `lhc_web/lib` - Core of the application
  * `lib/core` - Service classes and business logic
  * `lib/models` - Database model classes using erLhcoreClassDBTrait
  * `lib/vendor_lhc` - Dynamically loaded models and classes
* `lhc_web/lhcore_autoload.php` - Main application autoload file
* `lhc_web/modules` - Application modules organized by feature (lhchat, lhuser, lhdepartment, etc.)
* `lhc_web/pos` - Persistent Object definitions (ORM field type mappings)
* `lhc_web/settings` - Configuration files
* `lhc_web/translations` - Application translations

# Database structure reference

See: `lhc_web/doc/update_db/structure.json` for complete database schema
See: `.github/instructions/10-summary.json` for primary entities overview

# MVC Pattern workflow

## Where are models defined?

* Models classes are defined `lhc_web/lib/models` 
* Models classes with dynamic autoload are defined at `lhc_web/lib/vendor_lhc/LiveHelperChat/Models`
* `pos` files for their models for types of fields are defined in `lhc_web/pos` 
* `pos` files or classes with dynamic autoload are defined in `lhc_web/pos/lhabstract/livehelperchat/models` 

## Database Layer and ORM

* All model classes use the `erLhcoreClassDBTrait` trait located at `lhc_web/lib/core/lhcore/lhdbtrait.php`
* This trait provides common database operations like save, update, delete, and query functionality
* Models must define static properties: `$dbTable`, `$dbTableId`, `$dbDefaultSort`, `$dbSortOrder`
* Database session handler is configured via static properties `$dbSessionHandler` and `$dbSessionHandlerUrl`

### Common Database Operations

* `setState($properties)` - Set multiple object properties from array
* `saveThis($params)` - Save or update record with lifecycle hooks
* `saveThisOnly($params)` - Save new record only
* `saveOrUpdate($params)` - Alias for saveThis
* `updateThis($params)` - Update existing record
* `removeThis()` - Delete record
* `syncAndLock($columns)` - Lock record and sync columns from database
* `refreshThis()` - Refresh object from database
* `clearCache()` - Clear cached data for object
* `getFields()` - Get field definitions from PHP include file
* `fetch($id, $useCache, $throwException)` - Load single record by ID
* `fetchAndLock($id, $useCache)` - Load single record by ID with database lock
* `fetchCache($id)` - Load with memcache support
* `isOwner($id, $skipChecking)` - Check if current user owns object
* `findOne($params)` - Find single record with conditions
* `getList($params)` - Get list of records with filtering/sorting
* `getCount($params, $operation, $field, $rawSelect, $fetchColumn, $fetchAll, $fetchColumnAll, $groupedCount)` - Count or aggregate records
* `estimateRows()` - Get estimated table row count from information_schema

### Query Parameters

* `filter` - Exact match conditions
  ```php
  // Example: Get chats with specific status
  $chats = erLhcoreClassModelChat::getList(array(
      'filter' => array('status' => 1)
  ));
  ```
* `filterfields` - Multiple filter combinations
  ```php
  // Example: Multiple filter combinations for different conditions
  $chats = erLhcoreClassModelChat::getList(array(
      'filterfields' => array(
          array('status' => 1, 'dep_id' => 5),
          array('status' => 2, 'dep_id' => 10)
      )
  ));
  ```
* `filterin` - IN clause conditions
  ```php
  // Example: Get chats from multiple departments
  $chats = erLhcoreClassModelChat::getList(array(
      'filterin' => array('dep_id' => array(1, 2, 3, 4))
  ));
  ```
* `filterinfields` - Multiple IN clause combinations
  ```php
  // Example: Multiple IN clause combinations for different conditions
  $chats = erLhcoreClassModelChat::getList(array(
      'filterinfields' => array(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LiveHelperChat/livehelperchat](https://github.com/LiveHelperChat/livehelperchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
