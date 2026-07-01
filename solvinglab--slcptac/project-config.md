---
trigger: always_on
description: **Package Type**: Proteogenomic Analysis Toolkit (Protein/Phospho + Genomics + Clinical)
---

# SLCPTAC Documentation Rules for LLM RAG System

**Package Type**: Proteogenomic Analysis Toolkit (Protein/Phospho + Genomics + Clinical)  
**Target Users**: Cancer Researchers + Proteomics Scientists + LLM Systems + Cursor AI Agent  
**Core Purpose**: Enable LLM to SELECT the right analysis scenario and GENERATE correct proteogenomic analysis calls

**CRITICAL DISTINCTION**: SLCPTAC = CPTAC Proteomics + Phosphoproteomics (NOT TCGA)

---

## 🎯 CORE UNDERSTANDING: Proteogenomic Analysis Package

**SLCPTAC = 17 Statistical Scenarios + Phosphorylation-Centric Analysis**

```
User Query: "AKT1蛋白水平和磷酸化之间有什么关系?"
    ↓
LLM RAG Retrieval → Finds cptac_correlation()
    ↓
LLM Reads: @description + @param + @examples
    ↓
LLM Generates: result <- cptac_correlation(
                  var1 = "AKT1",
                  var1_modal = "Protein",     # Protein abundance
                  var1_cancers = "BRCA",
                  var2 = "AKT1",
                  var2_modal = "Phospho",     # Auto-detects all phospho sites
                  var2_cancers = "BRCA"
                )
    ↓
Execute → Returns: Correlation for each phospho site + Heatmap
    ↓
LLM Answers: "AKT1蛋白水平与其9个磷酸化位点正相关，
             S473位点相关性最强 (r=0.68, p<0.001)"
```

**Unique SLCPTAC Features**:
- ✅ **Proteomics**: Protein abundance (mass spectrometry)
- ✅ **Phosphoproteomics**: Site-specific phosphorylation levels
- ✅ **Auto-site detection**: Input "AKT1" → Returns all phospho sites
- ✅ **Transcriptome-Proteome integration**: Compare mRNA vs protein
- ✅ **Mutation-Phospho association**: How mutations affect phosphorylation

**Critical Difference from Other Packages**:
- ❌ NOT: Raw data analysis (like DoCCI, RunPTA)
- ✅ YES: Pre-defined statistical scenarios on curated datasets
- Users ask **research questions**, not **how to analyze data**

---

## 🚨 FIVE CORE PRINCIPLES (NON-NEGOTIABLE)

### Principle #1: TEST-FIRST (测试成功是基石)

**For statistical functions, testing = Scenario execution + Result verification**

```r
# Test template for SLTCGA/SLCPTAC functions
Rscript -e "
library(SLTCGA)  # or SLCPTAC
start <- Sys.time()

# Test with real research question
result <- tcga_correlation(
  var1 = 'TP53', var1_modal = 'RNAseq', var1_cancers = 'BRCA',
  var2 = 'MDM2', var2_modal = 'RNAseq', var2_cancers = 'BRCA',
  method = 'pearson'
)

runtime <- as.numeric(difftime(Sys.time(), start, units = 'secs'))
cat('✓ Analysis successful\n')
cat('Runtime:', runtime, 'sec\n')
cat('Correlation:', result$statistics$correlation, '\n')
cat('P-value:', result$statistics$pvalue, '\n')
cat('Plot saved:', result$plot_path, '\n')
"
```

**What to Document from Test**:
- ✅ Analysis runtime (typically 1-30 sec depending on scenario)
- ✅ Result structure (statistics, plot, data)
- ✅ Statistical values (correlation, p-value, HR, etc.)
- ✅ Plot output (file path)

**If test fails**:
- Data loading error → Check if `SL_BULK_DATA` environment variable is set
- Scenario mismatch → Check if variable types match the scenario
- STOP and report to user

---

### Principle #2: ENGLISH ONLY

- ✅ ALL roxygen documentation in English
- ✅ ALL code comments in English
- ❌ NO Chinese characters: 相关分析, 富集分析, 生存分析, etc.

---

### Principle #3: DELETE ALL NOISE

**Category 1: Data Setup (MINIMIZE)**

❌ DELETE verbose instructions:
```r
#' **Data Setup**:
#' \itemize{
#'   \item Step 1: Download TCGA data from...
#'   \item Step 2: Set environment variable SL_BULK_DATA...
#'   \item Step 3: Preprocess data using...
#' }
```

✅ KEEP minimal one-liner:
```r
#' **Data**: Requires \code{Sys.setenv(SL_BULK_DATA = "/path/to/data")}. 
#' See \code{vignette("setup")} for first-time configuration.
```

**Category 2: Statistical Jargon (KEEP if necessary for interpretation)**

✅ Keep statistical context:
```r
#' @details
#' **Interpreting Results**:
#' \itemize{
#'   \item **Pearson correlation**: Measures linear relationship (-1 to 1)
#'   \item **p-value**: Statistical significance (< 0.05 typically significant)
#'   \item **Hazard ratio**: >1 indicates increased risk, <1 protective effect
#' }
```

**Category 3: Filler Words (DELETE)**

❌ "Powerful analysis", "Comprehensive toolkit", "Advanced statistics"

---

### Principle #4: REFERENCE-BACKED

Every statistical method MUST have:
- Method paper citation
- DOI link
- Statistical test reference (if non-standard)

**Search priority**:
1. Check existing @references in the .R file
2. Search for method name in documentation
3. Use web_search: "[Method name] statistical test paper DOI"
4. If not found → Ask user

---

### Principle #5: EXECUTABLE EXAMPLES

- ✅ ALL @examples must use \donttest{} (not \dontrun{})
- ✅ Examples must use real cancer types and genes
- ✅ Show expected output structure
- ✅ Include interpretation guidance

---

## 📋 DOCUMENTATION STRUCTURE FOR STATISTICAL FUNCTIONS

### @title (Scenario-Oriented)

**Format**: `[Statistical Analysis Type] - [Data Types]`

✅ Good:
```r
#' Correlation Analysis Across Multi-Omics Data
#' Enrichment Analysis - Mutation-Driven Pathway Changes
#' Survival Analysis with Clinical and Molecular Variables
```

❌ Bad:
```r
#' TCGA Correlation  ← Too vague
#' Analyze Data  ← Not specific
```

---

### @description (3-4 sentences, 60-100 words)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SolvingLab/SLCPTAC](https://github.com/SolvingLab/SLCPTAC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
