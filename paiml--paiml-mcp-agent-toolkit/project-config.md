---
trigger: always_on
description: Validates: capability claims against AST, file path references, function/module references, external URLs (404 detection). Uses Semantic Entropy (Nature 2024) and MIND framework (IJCAI 2025).
---

# Claude Code Configuration

## CRITICAL: Sovereign AI Dependency Policy (80/20 Batuta Stack)

**MANDATORY: Minimize external dependencies - use batuta stack first**

PMAT follows the Sovereign AI philosophy: **80% batuta stack, 20% external deps maximum**.

Before adding ANY external dependency for math, algorithms, data science, ML, or compute:
1. **CHECK BATUTA STACK FIRST** - See if sovereign tools already provide the functionality
2. **BUILD IF CLOSE** - If batuta stack is 70%+ there, extend it rather than adding external dep
3. **EXTERNAL ONLY AS LAST RESORT** - Document why batuta stack couldn't work

### Batuta Stack (Sovereign AI Tools)

| Crate | Purpose | Use Instead Of |
|-------|---------|----------------|
| `aprender` | ML, stats, graph algorithms, text similarity | nalgebra, linfa, smartcore |
| `trueno` | SIMD/GPU compute, matrix ops | ndarray, nalgebra |
| `trueno-graph` | Graph database, PageRank, Louvain | petgraph, graph |
| `trueno-db` | Columnar storage, analytics | polars, datafusion |
| `trueno-rag` | RAG pipeline, vector search | qdrant, milvus |
| `trueno-viz` | Terminal visualization | plotters, textplots |
| `trueno-zram-core` | SIMD compression | lz4, zstd |
| `renacer` | Golden tracing, chaos testing | proptest chaos |
| `certeza` | Quality validation | custom scripts |
| `bashrs` | Bash/Makefile linting | shellcheck |
| `probar` | Property-based testing | quickcheck |
| `pmcp` | MCP protocol SDK | custom MCP |
| `presentar-core` | TUI framework | ratatui |

### Dependencies Requiring Review

| External Dep | Status | Batuta Alternative |
|--------------|--------|-------------------|
| `nalgebra-sparse` | Review | `aprender::primitives` sparse matrices |
| `roaring` | Keep | Specialized bitmap (no batuta equivalent yet) |
| `rand` | Keep | Foundational (may add to trueno later) |
| `rayon` | Keep | Foundational parallel iterator |

Before adding ANY new dependency: check batuta stack first (`pmat query "YourFeature" --limit 5` in aprender), build if close, document if external required.

---

## CRITICAL: Code Search Policy

**NEVER use grep/glob for code search. ALWAYS use `pmat query`.**

| Task | Command |
|------|---------|
| Find functions by intent | `pmat query "error handling" --limit 10` |
| Find high-quality examples | `pmat query "serialize" --min-grade A` |
| Find simple implementations | `pmat query "cache" --max-complexity 10` |
| Find important functions | `pmat query "dispatch" --rank-by pagerank` |
| Find with fault patterns | `pmat query "unwrap" --faults --exclude-tests` |
| Cross-project search | `pmat query "simd" --include-project ../trueno` |
| Include source code | `pmat query "tokenize" --include-source` |
| Search by commit intent | `pmat query "fix memory leak" -G` |
| Find volatile hot code | `pmat query "cache" --churn` |
| Find code clones | `pmat query "serialize" --duplicates` |
| Find repetitive patterns | `pmat query "handler" --entropy` |
| Full enrichment | `pmat query "dispatch" --churn --duplicates --entropy --faults -G` |
| Regex search (like rg -e) | `pmat query --regex "fn\s+handle_\w+" --limit 10` |
| Literal string search (like rg -F) | `pmat query --literal "unwrap()" --limit 10` |
| Exclude pattern (like grep -v) | `pmat query "handler" --exclude "test"` |
| Exclude files by glob | `pmat query "cache" --exclude-file "tests"` |
| Case-insensitive search | `pmat query "Error" -i` |
| Files with matches (like rg -l) | `pmat query "handler" --files-with-matches` |
| Count matches per file (like rg -c) | `pmat query "unwrap" --count` |
| Context lines (like grep -C) | `pmat query "panic" -A 3 -B 2` |

### Search Mode Flags

- **`--regex`** — Regex pattern matching against function name, signature, and source. Uses Rust regex syntax.
- **`--literal`** — Exact literal string match (no semantic ranking). Like `rg -F`.
- **`--case-sensitive`** — Force case-sensitive matching (default: smart-case like rg).
- **`-i` / `--ignore-case`** — Force case-insensitive matching.
- **`--exclude PATTERN`** — Exclude results matching content pattern (like `grep -v`).
- **`--exclude-file GLOB`** — Exclude results from files matching glob pattern.
- **`--files-with-matches`** — Output only unique file paths (like `rg -l`).
- **`--count`** — Output match count per file (like `rg -c`).
- **`-A N` / `-B N` / `-C N`** — Show N lines of context after/before/around matches.

### Enrichment Flags

- **`-G` / `--git-history`** — Fuse git commit history via RRF. Finds code by intent via commit message semantic search.
- **`--churn`** — Git volatility metrics (90-day window). Hot files (>50% churn) flagged.
- **`--duplicates`** — Code clone detection via MinHash + LSH.
- **`--entropy`** — Pattern diversity metrics. Low (<30%) = boilerplate; high (>80%) = unique.
- **`--faults`** — Batuta fault pattern annotations (unwrap, panic, unsafe, etc.).

### Coverage Gap Analysis

**MANDATORY: Use `pmat query --coverage-gaps` for coverage work. NEVER use `make coverage` or raw `cargo llvm-cov` output.**

```bash
# Find top coverage gaps ranked by uncovered lines
pmat query --coverage-gaps --limit 30 --exclude-tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paiml/paiml-mcp-agent-toolkit](https://github.com/paiml/paiml-mcp-agent-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
