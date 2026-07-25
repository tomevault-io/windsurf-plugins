---
trigger: always_on
description: **curatedMetagenomicData** is a Bioconductor R package providing standardized, curated human microbiome data. It curates metagenomic abundance data (gene families, marker presence/abundance, pathway coverage/abundance, relative abundance) processed through MetaPhlAn3 and HUMAnN3, returning data as SummarizedExperiment or TreeSummarizedExperiment objects.
---

# AI Coding Agent Instructions for curatedMetagenomicData

## Project Overview
**curatedMetagenomicData** is a Bioconductor R package providing standardized, curated human microbiome data. It curates metagenomic abundance data (gene families, marker presence/abundance, pathway coverage/abundance, relative abundance) processed through MetaPhlAn3 and HUMAnN3, returning data as SummarizedExperiment or TreeSummarizedExperiment objects.

## Architecture & Data Flow

### Three-Function User API
The package exports three main functions (`R/`):
1. **`curatedMetagenomicData(pattern, dryrun=TRUE, counts=FALSE, rownames="long")`** - Query & retrieve data by regex pattern against ExperimentHub resource titles
   - Returns invisibly when `dryrun=TRUE` (resource list), or a named `list` of SE/TSE objects when `FALSE`
   - Handles date selection (picks most recent resource when multiple dates exist)
   - Special handling for `relative_abundance`: can convert proportions to counts via `read_depth` multiplication

2. **`mergeData(mergeList)`** - Merge returned list elements across studies (same dataType only)
   - Returns TreeSummarizedExperiment for `relative_abundance`, SummarizedExperiment otherwise
   - Uses dplyr joins on assays/rowData; memory-intensive for many elements

3. **`returnSamples(sampleMetadata, dataType, counts, rownames)`** - Convenience wrapper: filters samples using subset sampleMetadata, retrieves resources, merges, and subsets result

### Data Structure
- **Resource Titles** (`R/sysdata.rda`): Embedded vector of all available resource names matching pattern `YYYY-MM-DD.StudyName.dataType`
- **Sample Metadata** (`data/sampleMetadata.rda`): User-facing DataFrame for browsing/filtering (derived from curatedMetagenomicDataCuration package)
- **ExperimentHub**: Sparse matrices stored here; package queries via `ExperimentHub::ExperimentHub()` + pattern matching

#### Sparse Matrix Storage Optimization
Only `gene_families` dataType is stored as sparse matrices in ExperimentHub to reduce cloud storage footprint. Other datatypes are dense matrices. When `curatedMetagenomicData()` retrieves resources, it constructs SummarizedExperiment objects on-the-fly by wrapping these matrices with corresponding sample metadata from `colData` and feature annotations from `rowData`. This lazy-loading approach means minimal local disk space is consumed until users explicitly load data.

#### metadata.csv and resourceTitles Relationship
- **[inst/extdata/metadata.csv](inst/extdata/metadata.csv)** is the single source of truth for all available resources. Each row represents one ExperimentHub resource with columns: `Title` (resource name), `Description`, `BiocVersion`, etc.
- **resourceTitles** (embedded in `R/sysdata.rda`) is a character vector of all `Title` values extracted from metadata.csv. Used for fast in-memory pattern matching when users call `curatedMetagenomicData(pattern)`.
- **Generation pipeline**: Dated CSVs in `inst/extdata/` (one per metadata update) → combined into metadata.csv via [inst/scripts/make-metadata.R](inst/scripts/make-metadata.R) → resourceTitles regenerated from metadata.csv via [data-raw/resourceTitles.R](data-raw/resourceTitles.R).
- Tests validate consistency: [test-curatedMetagenomicData.R](tests/testthat/test-curatedMetagenomicData.R#L10-L30) asserts that all titles in metadata.csv match returned resourceTitles.

### Key Dependencies
- **SummarizedExperiment/TreeSummarizedExperiment**: S4 container classes for assay data + metadata
- **dplyr/tidyr**: Heavy use for joining assays/metadata (see NAMESPACE for 20+ imports)
- **ExperimentHub/AnnotationHub**: Cloud data access via pattern queries
- **MetaPhlan3/HUMAnN3**: External tools that preprocessed the data (not in package)

## Developer Workflows

### Test Execution
Run tests with `R CMD check` or in R console:
```R
devtools::test()  # or testthat::test_dir("tests/testthat")
```
Key test files:
- [tests/testthat/test-curatedMetagenomicData.R](tests/testthat/test-curatedMetagenomicData.R): Tests pattern matching, resource existence, return types
- [tests/testthat/test-mergeData.R](tests/testthat/test-mergeData.R): Tests merging logic
- Asserts against [inst/extdata/metadata.csv](inst/extdata/metadata.csv) (single source of truth)

### Data Update Workflow
When adding new studies (uncommon):
1. **Generate metadata CSV**: [inst/scripts/make-metadata.R](inst/scripts/make-metadata.R) - Combines dated CSVs in `inst/extdata/` into single metadata.csv
2. **Update sampleMetadata**: [data-raw/sampleMetadata.R](data-raw/sampleMetadata.R) - Pulls from curatedMetagenomicDataCuration package, validates against resourceTitles, saves as .rda
3. **Regenerate resource titles**: [data-raw/resourceTitles.R](data-raw/resourceTitles.R) - Extracts from metadata.csv, saved to sysdata.rda
4. **Rebuild documentation**: Run `roxygen2::roxygenise()` to regenerate NAMESPACE & .Rd files

### Documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [waldronlab/curatedMetagenomicData](https://github.com/waldronlab/curatedMetagenomicData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
