---
trigger: always_on
description: **Purpose:** Calculate and compare PAF alignment statistics, especially useful for validating filtering operations.
---

# SweepGA Filtering Algorithm

## Tools Available

### alnstats - Alignment Statistics Tool

**Purpose:** Calculate and compare PAF alignment statistics, especially useful for validating filtering operations.

**Usage:**
```bash
# Show statistics for a PAF file
alnstats alignments.paf

# Compare before/after filtering
alnstats raw.paf filtered.paf

# Detailed per-genome-pair breakdown
alnstats alignments.paf -d
```

**Key Metrics:**
- Total mappings, bases, average identity
- Per-genome-pair coverage statistics
- Genome pairs with >95% coverage
- Inter-chromosomal and inter-genome mapping counts

**When to Use:**
1. **After filtering operations** - Verify genome pairs aren't lost
2. **Before/after comparisons** - Quantify filtering effects
3. **Debugging coverage issues** - Use `-d` to see per-pair breakdown
4. **Validating changes** - Ensure modifications preserve expected coverage

**Example validation workflow:**
```bash
# Before making changes
alnstats z.paf > before_stats.txt

# Make changes, rerun filtering
sweepga z.paf > filtered.paf

# Compare results
alnstats z.paf filtered.paf

# Verify all genome pairs preserved
alnstats filtered.paf -d | grep "cov"
```

## CRITICAL: Git Commit Rules - NEVER VIOLATE THESE

**ABSOLUTELY FORBIDDEN - NO EXCEPTIONS:**
- **NEVER EVER use `git add -A`, `git add .`, or `git add --all`**
- **NEVER add test data, FASTA files (except data/scerevisiae8.fa.gz), PAF files, or intermediate results**
- **NEVER add files with extensions: .fa, .fasta, .paf, .gdb, .ktab, .bps, .log (unless specifically code logs)**

**REQUIRED PROCEDURE:**
1. Always list files to be added explicitly: `git add src/specific_file.rs`
2. Always check with `git status` before committing
3. If you accidentally stage wrong files, immediately `git reset HEAD <file>`
4. All test data must be generated from data/scerevisiae8.fa.gz only

**BEFORE EVERY COMMIT:**
```bash
git status  # Check what's staged
git diff --cached --name-only  # List all staged files
# Only proceed if list contains ONLY source code files
```

## CRITICAL FIX: 1:1 Filtering with ~100% Coverage (SOLVED)

### The Problem
When applying 1:1 filtering to 99% identical yeast genomes, we were only getting ~23% coverage instead of the expected ~99%. This was unacceptable for genome alignment where we need complete coverage between highly similar genomes.

### The Solution
The key insight is that **1:1 filtering must operate at the chromosome pair level, not the genome pair level**.

#### What Was Wrong:
- We were grouping by genome prefix pairs (e.g., "SGDref#1" → "DBVPG6765#1")
- Within each genome pair group (~460 alignments across all chromosomes), we only kept 1 alignment total
- This meant only 1 chromosome pair got an alignment per genome pair = terrible coverage

#### The Fix:
- Group by full chromosome names (e.g., "SGDref#1#chrI" → "DBVPG6765#1#chrI")
- Apply 1:1 filtering within each chromosome pair group
- Use `plane_sweep_both()` for true 1:1 (respecting both query and target constraints)
- Result: Keep the best alignment for EACH chromosome pair = ~100% coverage

### Implementation Details
In `paf_filter.rs`:
```rust
// For 1:1 filtering: group by full chromosome names (includes genome prefix)
let query_group = meta.query_name.clone();
let target_group = meta.target_name.clone();
```

In `plane_sweep_exact.rs`:
```rust
// True 1:1 filtering applies constraints on BOTH axes
let kept_in_group = if mappings_to_keep == 1 {
    plane_sweep_both(&mut group_mappings, 1, 1, overlap_threshold)
} else {
    plane_sweep_query(&mut group_mappings, mappings_to_keep, overlap_threshold)
};
```

### Results
- Before fix: 1,738 alignments, 23.1% coverage, 0% genome pairs >95% coverage
- After fix: 26,272 alignments, 100.1% coverage, 100% genome pairs >95% coverage

This maintains the expected property that 99% identical genomes should have ~100% reciprocal coverage while still removing redundant/overlapping alignments within each chromosome pair.

## Core Algorithm (Corrected Implementation)

The filtering process follows this exact sequence:

1. **Input Processing**
   - Take input mappings/alignments from any source (PAF format)
   - Filter by minimum block length and optionally exclude self-mappings
   - Store all original mappings for potential rescue later

2. **Primary Mapping Filter** (default: 1:1)
   - Apply plane sweep filtering to raw mappings BEFORE scaffold creation
   - Respects PanSN prefix grouping when `-Y` is set
   - Controlled by `-n/--num-mappings` (default: "1:1")
   - Options: "1:1", "1" (same as "1:∞"), "N" (no filtering)

3. **Scaffold Creation** (if `-s` > 0)
   - Create scaffolds from the (optionally pre-filtered) mappings
   - Merge nearby mappings into scaffold chains (using `-j/--scaffold-jump` parameter, default: 10kb)
   - Filter chains by minimum length (`-s/--scaffold-mass`, default: 10kb)
   - Scaffolds define high-confidence syntenic regions

4. **Scaffold Filter** (default: 1:1)
   - Apply plane sweep filtering to the SCAFFOLD chains
   - Respects PanSN prefix grouping when `-Y` is set
   - Controlled by `-m/--scaffold-filter` (default: "1:1")
   - Options: "1:1", "1" (same as "1:∞"), "N" (no filtering)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pangenome/sweepga](https://github.com/pangenome/sweepga) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
