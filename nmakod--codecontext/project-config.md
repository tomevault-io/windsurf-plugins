---
trigger: always_on
description: **Generated:** 2025-08-31T18:16:48+05:30
---

# CodeContext Map

**Generated:** 2025-08-31T18:16:48+05:30  
**Version:** 2.0.0  
**Analysis Time:** 10.366986959s  
**Status:** Real Tree-sitter Analysis

## 📊 Overview

This context map was generated using **real Tree-sitter parsing** and provides comprehensive analysis of your codebase:

- **Files Analyzed**: 120 files
- **Symbols Extracted**: 2265 symbols  
- **Languages Detected**: 3 languages
- **Import Relationships**: 110 file dependencies

### 🎯 Analysis Capabilities
- ✅ **Real AST Parsing** - Tree-sitter JavaScript/TypeScript grammars
- ✅ **Symbol Extraction** - Functions, classes, methods, variables, imports
- ✅ **Dependency Analysis** - File-to-file relationship mapping
- ✅ **Multi-language Support** - TypeScript, JavaScript, JSON, YAML

## 📁 File Analysis

| File | Language | Lines | Symbols | Imports | Type |
|------|----------|-------|---------|---------|------|
| `.claude/settings.local.json` | json | 27 | 0 | 0 | source |
| `.codecontext/config.yaml` | yaml | 80 | 0 | 0 | source |
| `.github/dependabot.yml` | yaml | 23 | 0 | 0 | source |
| `.github/release.yml` | yaml | 36 | 0 | 0 | source |
| `.release-please-manifest.json` | json | 3 | 0 | 0 | source |
| `cmd/codecontext/main.go` | go | 24 | 3 | 1 | source |
| `internal/analyzer/graph.go` | go | 1261 | 45 | 1 | source |
| `internal/analyzer/graph_test.go` | go | 1532 | 48 | 1 | test |
| `internal/analyzer/incremental.go` | go | 589 | 24 | 1 | source |
| `internal/analyzer/incremental_test.go` | go | 680 | 15 | 1 | test |
| `internal/analyzer/markdown.go` | go | 748 | 34 | 1 | source |
| `internal/analyzer/relationships.go` | go | 486 | 22 | 1 | source |
| `internal/analyzer/relationships_test.go` | go | 473 | 12 | 1 | test |
| `internal/cache/persistent.go` | go | 597 | 37 | 1 | source |
| `internal/cache/persistent_test.go` | go | 526 | 14 | 1 | test |
| `internal/cli/compact.go` | go | 198 | 9 | 1 | source |
| `internal/cli/compact_test.go` | go | 129 | 4 | 1 | test |
| `internal/cli/generate.go` | go | 175 | 5 | 1 | source |
| `internal/cli/init.go` | go | 203 | 4 | 1 | source |
| `internal/cli/init_test.go` | go | 115 | 3 | 1 | test |
| `internal/cli/integration_test.go` | go | 799 | 31 | 1 | test |
| `internal/cli/mcp.go` | go | 119 | 4 | 1 | source |
| `internal/cli/mcp_test.go` | go | 656 | 11 | 1 | test |
| `internal/cli/progress.go` | go | 611 | 44 | 1 | source |
| `internal/cli/progress_test.go` | go | 500 | 30 | 1 | test |
| `internal/cli/root.go` | go | 86 | 6 | 1 | source |
| `internal/cli/shutdown.go` | go | 408 | 31 | 1 | source |
| `internal/cli/update.go` | go | 116 | 6 | 1 | source |
| `internal/cli/watch.go` | go | 584 | 27 | 1 | source |
| `internal/compact/controller.go` | go | 466 | 28 | 1 | source |
| `internal/compact/controller_test.go` | go | 715 | 20 | 1 | test |
| `internal/compact/strategy.go` | go | 859 | 49 | 1 | source |
| `internal/compact/strategy_test.go` | go | 698 | 23 | 1 | test |
| `internal/config/config.go` | go | 34 | 2 | 0 | source |
| `internal/diff/ast.go` | go | 641 | 59 | 1 | source |
| `internal/diff/dependency.go` | go | 781 | 88 | 1 | source |
| `internal/diff/engine.go` | go | 672 | 40 | 1 | source |
| `internal/diff/heuristics.go` | go | 641 | 51 | 1 | source |
| `internal/diff/rename.go` | go | 513 | 37 | 1 | source |
| `internal/diff/semantic.go` | go | 526 | 38 | 1 | source |
| `internal/diff/similarity.go` | go | 760 | 64 | 1 | source |
| `internal/diff/utils.go` | go | 39 | 4 | 0 | source |
| `internal/generator/markdown.go` | go | 69 | 5 | 1 | source |
| `internal/git/analyzer.go` | go | 354 | 25 | 1 | source |
| `internal/git/analyzer_test.go` | go | 425 | 14 | 1 | test |
| `internal/git/error_handling_test.go` | go | 119 | 15 | 1 | test |
| `internal/git/integration.go` | go | 1279 | 71 | 1 | source |
| `internal/git/integration_flow_test.go` | go | 402 | 6 | 1 | test |
| `internal/git/integration_test.go` | go | 779 | 27 | 1 | test |
| `internal/git/interfaces.go` | go | 23 | 3 | 1 | source |
| `internal/git/pattern_detection_integration_test.go` | go | 424 | 9 | 1 | test |
| `internal/git/patterns.go` | go | 667 | 34 | 1 | source |
| `internal/git/patterns_ignore_test.go` | go | 188 | 5 | 1 | test |
| `internal/git/patterns_test.go` | go | 341 | 13 | 1 | test |
| `internal/git/performance_benchmark_test.go` | go | 231 | 17 | 1 | test |
| `internal/git/semantic.go` | go | 495 | 31 | 1 | source |
| `internal/git/semantic_analysis_e2e_test.go` | go | 265 | 17 | 1 | test |
| `internal/git/semantic_test.go` | go | 477 | 14 | 1 | test |
| `internal/git/simple_patterns.go` | go | 219 | 14 | 1 | source |
| `internal/git/simple_patterns_test.go` | go | 457 | 11 | 1 | test |
| `internal/mcp/migration_test.go` | go | 93 | 5 | 1 | test |
| `internal/mcp/server.go` | go | 1343 | 50 | 1 | source |
| `internal/mcp/server_test.go` | go | 1040 | 22 | 1 | test |
| `internal/parser/builder.go` | go | 266 | 14 | 1 | source |
| `internal/parser/cache.go` | go | 194 | 17 | 1 | source |
| `internal/parser/cache_test.go` | go | 300 | 10 | 1 | test |
| `internal/parser/config.go` | go | 111 | 4 | 1 | source |
| `internal/parser/cpp_framework_test.go` | go | 324 | 3 | 1 | test |
| `internal/parser/cpp_integration_test.go` | go | 534 | 5 | 1 | test |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nmakod/codecontext](https://github.com/nmakod/codecontext) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
