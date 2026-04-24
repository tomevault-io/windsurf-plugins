---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WeRelate.org is a genealogical wiki built on MediaWiki. The codebase extends MediaWiki with custom structured namespaces for genealogical data (Person, Family, Place, Source, etc.) and specialized functionality for family tree visualization, GEDCOM import/export, and genealogical data management.

## Setup and Installation

### Database Setup
```bash
# Download and import skeleton database
wget http://public.werelate.org.s3.amazonaws.com/wikidb.sql
mysql -u [user] -p [database_name] < wikidb.sql
```

### Simple Setup
```bash
cp htaccess.sample .htaccess
# Edit environment variables in .htaccess
```

### Full Setup
1. Copy files to a `w` directory under your htdocs root
2. Create Apache site file from `conf/apache2.sample`
3. Copy to `apache2/sites-available/[sitename]`
4. Fill in/modify values based on your environment
5. Enable the site: `a2ensite [sitename]`

### Running Jobs
Background jobs are disabled in the web context (`$wgJobRunRate = 0`). Run jobs manually:
```bash
php maintenance/runJobs.php
```

## Architecture

### Core MediaWiki
- **Entry point**: `index.php` - Main wiki script that initializes MediaWiki and dispatches requests
- **Configuration**: `LocalSettings.php` - Site configuration (uses environment variables from .htaccess or Apache config)
- **Core includes**: `includes/` - MediaWiki core files (Article, Parser, Database, etc.)

### Custom Namespaces (Structured Data)
Located in `extensions/structuredNamespaces/`:

All structured namespaces extend `StructuredData.php` which provides:
- XML parsing and validation
- Edit form rendering and data import
- Propagation of edits across linked pages
- Data quality checking via `DQHandler.php`

**Note**: Extension loading order in LocalSettings.php matters. Source and UserPage must be loaded before Name and Place because they contain names and places.

**Namespace Constants** (defined in LocalSettings.php):
| Namespace | Constant | ID |
|-----------|----------|-----|
| Givenname | NS_GIVEN_NAME | 100 |
| Surname | NS_SURNAME | 102 |
| Source | NS_SOURCE | 104 |
| Place | NS_PLACE | 106 |
| Person | NS_PERSON | 108 |
| Family | NS_FAMILY | 110 |
| MySource | NS_MYSOURCE | 112 |
| Repository | NS_REPOSITORY | 114 |
| Portal | NS_PORTAL | 116 |
| Transcript | NS_TRANSCRIPT | 118 |

**Key structured namespace classes:**
- `Person.php` - Individual person pages with birth/death/events
- `Family.php` - Family pages linking spouses and children
- `Place.php` - Geographic locations with coordinates
- `Source.php` - Citations and sources
- `Name.php` - Handles Givenname and Surname namespaces
- `MySource.php` - User-specific sources
- `Repository.php` - Archive/repository information
- `Transcript.php` - Historical document transcriptions
- `UserPage.php` - Enhanced user pages
- `ArticlePage.php` - Enhanced article pages
- `SDImage.php` - Image handling with structured data

**Supporting infrastructure:**
- `ESINHandler.php` - Event, Source, Image, and Name data structure handling
- `PropagationManager.php` - Manages cascading edits when pages are moved/deleted/updated
- `DateHandler.php` - Genealogical date parsing and validation
- `DQHandler.php` - Data quality validation
- `TipManager.php` - Contextual help/tips for edit forms

### Family Tree Features
Located in `extensions/familytree/`:
- `FamilyTree.php` - Core family tree data structure and visualization
- `FamilyTreePropagator.php` - Updates family tree when pages change
- `SpecialShowFamilyTree.php` - Family tree visualization page
- `SpecialTreeRelated.php` - Find related people across trees
- `SpecialTreeDeletionImpact.php` - Analyze impact before deleting pages
- `SpecialTreeCountWatchers.php` - Track page watchers
- `SpecialCopyTree.php` - Copy family tree branches

### GEDCOM Import/Export
Located in `extensions/gedcom/`:
- `SpecialGedcomPage.php` - GEDCOM file upload and processing
- `SpecialGedcoms.php` - List and manage GEDCOM imports
- `GedcomExportJob.php` - Background job for GEDCOM export
- `GedcomAjaxFunctions.php` - AJAX handlers for GEDCOM operations

GEDCOM directories (configured in LocalSettings.php):
- `$wrGedcomUploadDirectory` - Uploaded GEDCOM files
- `$wrGedcomInprocessDirectory` - Files being processed
- `$wrGedcomXMLDirectory` - Parsed XML data
- `$wrGedcomExportDirectory` - Generated exports
- `$wrGedcomArchiveDirectory` - Archived files

### Special Pages
Located in `extensions/other/`:
- `SpecialSearch.php` - Custom search (uses external search service)
- `SpecialDashboard.php` - User dashboard
- `SpecialBrowse.php` - Browse pages by namespace
- `SpecialAddPage.php` - Add new pages with validation
- `SpecialCompare.php` - Compare pages for merging
- `SpecialMerge.php` - Merge duplicate pages
- `SpecialShowDuplicates.php` - Find potential duplicates
- `SpecialReviewMerge.php` - Review merge suggestions
- `SpecialDataQuality.php` - Data quality reports
- `SpecialDQStats.php` - Data quality statistics
- `SpecialTrees.php` - Tree management
- `SpecialNetwork.php` - Social network features
- `SpecialRequestDelete.php` - Request page deletion

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/werelate) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
