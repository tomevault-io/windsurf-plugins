---
trigger: always_on
description: This project implements automated weekly updates of STAC DEM BC JSONs using VM-based cron automation with incremental change detection. The implementation adopts proven performance improvements from stac_orthophoto_bc (parallel processing, pre-validation) before building automation infrastructure.
---

# CLAUDE.md - STAC DEM BC Project Guidelines

## Project Overview: Automated Weekly STAC DEM BC Updates

This project implements automated weekly updates of STAC DEM BC JSONs using VM-based cron automation with incremental change detection. The implementation adopts proven performance improvements from stac_orthophoto_bc (parallel processing, pre-validation) before building automation infrastructure.

**Architecture:** VM-based cron → Change detection → Parallel validation/processing → S3 sync → PgSTAC registration

**Expected Performance:**
- First run (full): ~1-1.5 hours (down from 5-6 hours)
- Weekly runs (incremental): 5-15 minutes for typical 10-50 new files
- Cost: $0 additional (uses existing VM)

### Key Implementation Phases

**Phase 1-2: Modernization ✅ COMPLETE (phase1-2-modernization worktree)**
- Port stac_orthophoto_bc performance improvements
- Pre-validation system with COG detection
- Parallel item creation using ThreadPoolExecutor
- Incremental update logic with change detection
- Optimize spatial extent calculation
- **Result:** 100-item test passed, ready for VM automation

**Phase 3: VM Automation (phase3-automation worktree - future)**
- Master automation script (stac_update_weekly.sh)
- Cron configuration on stac-prod VM
- Benchmarking and monitoring system
- Logging infrastructure

### Project Context

**Dataset:** 58,109 DEM GeoTIFFs from BC provincial objectstore (nrs.objectstore.gov.bc.ca/gdwuts)
- Grew 158% from initial 22,548 files (discovered in Phase 2.1 change detection)
- ~90 files with parentheses in filename excluded (all fail validation - see issue #8)

**Actual Performance (Feb 2026 - Full Build):**
- 58,028 items created in ~5.5 hours (~6,450 items/hour)
- Validation caching working (cache fix applied)
- Parallel processing with 32 workers
- 99.86% success rate (81 items failed/missing)
- **Bottleneck:** Network I/O reading remote GeoTIFFs for metadata

**Current Status:**
- ✅ Incremental update capability (change detection working)
- ✅ Validation caching (GeoTIFF validation)
- ✅ STAC JSON validation layer (new)
- ⏳ Manual execution (automation planned - Phase 3)
- ✅ Spatial extent optimized (hardcoded BC bbox)

**Goals:**
1. ~~Reduce full processing time to ~1-1.5 hours~~ → **Reality: 5-6 hours** (network I/O limited)
2. Enable weekly/monthly incremental updates (likely 30-60 min for 50-100 new files)
3. ✅ Implement robust validation and error handling
4. ⏳ Automate via VM cron jobs (Phase 3)
5. ✅ Maintain audit trail and benchmarking

**Key Learning:** Performance is network I/O bound, not CPU bound. Future optimization: local metadata caching (Issue #10).

### Related Work
- **stac_orthophoto_bc:** Reference implementation for parallel processing patterns
- **stac_uav_bc:** VM deployment patterns and automation functions
- **Issue #3:** Proper GeoTIFF validation and media type assignment

### Data Tracking & Validation System

**File-based tracking for quality assurance and incremental updates:**

```
data/
├── urls_list.txt              # Master URL list from BC objectstore (58,109 URLs)
├── urls_new.txt               # New URLs detected by change detection
├── urls_deleted.txt           # Deleted URLs (audit trail)
├── stac_geotiff_checks.csv    # Source validation (url, is_geotiff, is_cog)
└── stac_item_validation.csv   # Output validation (item_id, json_valid, error)
```

**Validation layers:**
1. **GeoTIFF validation** (`stac_geotiff_checks.csv`) - Validates source data quality
   - Checks if URL is readable GeoTIFF
   - Detects Cloud-Optimized GeoTIFF status
   - Caches results to avoid re-validation
   - Used during item creation to skip invalid sources

2. **STAC JSON validation** (`stac_item_validation.csv`) - Validates output data quality
   - Checks generated STAC item JSONs are valid
   - Uses pystac for spec compliance
   - Tracks validation errors for debugging
   - Filters items before PgSTAC registration
   - Script: `scripts/item_validate.py`

**Workflow integration:**
```
Source URLs → GeoTIFF Validation → Item Creation → JSON Validation → Registration
 (urls_list)   (geotiff_checks)      (.qmd/.py)    (item_validation)   (pgstac)
```

**Key insight:** Separation of source quality (can we read it?) from output quality (is STAC valid?) enables better debugging and incremental processing.

### Script Evolution: .qmd → .py

**Current state:**
- `.qmd` files: Good for exploration, mixed R/Python workflows
- `.py` scripts: Better for production, automation, testing

**Migration strategy (Issue #7):**
- New scripts: Write as pure Python (`.py`)
- Existing `.qmd`: Migrate gradually to standalone scripts
- Keep `.qmd`: For documentation/examples if useful

**Benefits of .py for production:**
- Better IDE support and debugging
- Easier testing and CI/CD integration
- Cleaner for cron/automation
- Standard Python packaging and distribution
- No R dependency for core workflows

### SRED Tracking
- Primary: https://github.com/NewGraphEnvironment/sred-2025-2026/issues/8

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NewGraphEnvironment) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
