---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LCRE (LimaCharlie Reverse Engineering) is a CLI tool for static binary analysis and forensics automation. It provides fast triage via native Go parsing for PE/ELF/Mach-O binaries, deep analysis via Ghidra headless integration, and enrichment from external analysis tools (capa, diec, floss, etc.). The tool is designed for AI assistant integration with machine-readable output formats.

## Build and Test Commands

```bash
make build          # Build binary to bin/lcre
make test           # Run all tests with coverage
make test-race      # Run tests with race detector (requires CGO_ENABLED=1)
make test-short     # Run quick tests only
make lint           # Run golangci-lint
make fmt            # Format code
make vet            # Run go vet
make dev            # Format, vet, test, and build
```

Run a single test:
```bash
go test -v -run TestFunctionName ./internal/package/...
```

## Architecture

### Two-Tier Analysis System

1. **Native Backend** (`internal/backend/native/`): Fast Go-native parsing for PE, ELF, and Mach-O binaries. Handles headers, imports, exports, sections, strings, and entropy calculation. Default backend.

2. **Ghidra Backend** (`internal/backend/ghidra/`): Deep analysis via headless Ghidra. Provides function extraction, decompilation, call graphs, and cross-references. Auto-triggered when Ghidra-specific commands are run.

### Backend Interface

All backends implement `backend.Backend` interface in `internal/backend/backend.go`. Backends self-register via init() to `backend.DefaultRegistry`.

### Caching Layer

`internal/cache/` provides SQLite-backed caching keyed by binary SHA256. Cache stores:
- Analysis results in SQLite (`analysis.db`)
- Quick-load metadata (`metadata.json`)
- Decompiled functions as `.c` files in `decompiled/` subdirectory

Cache location: `~/.cache/lcre/<sha256>/`

### CLI Structure

- `internal/cli/root.go`: Root command setup with global flags (`-o`, `-v`, `-t`)
- `internal/cli/analyze.go`: One-shot analysis command
- `internal/cli/query.go`: Parent for all cached query subcommands
- `internal/cli/query_*.go`: Individual query subcommands (summary, strings, imports, functions, decompile, etc.)

### Enrichment System

`internal/enrichment/` provides parsers for external tool output (e.g., from REMnux MCP).
The `lcre enrich` command imports tool results into the cache. Dedicated parsers exist for
capa, diec, and floss; all other tools have their raw output stored as-is.

- `internal/enrichment/enrichment.go`: Parser registry and `ParseToolOutput()` entry point
- `internal/enrichment/capa.go`: capa JSON -> capabilities with ATT&CK/MBC mappings
- `internal/enrichment/diec.go`: diec JSON -> packer/compiler detections
- `internal/enrichment/floss.go`: FLOSS JSON -> obfuscated/decoded strings
- `internal/cli/enrich.go`: `lcre enrich` command
- `internal/cli/query_capabilities.go`: `lcre query capabilities`
- `internal/cli/query_packer.go`: `lcre query packer`
- `internal/cli/query_enrichments.go`: `lcre query enrichments` / `lcre query enrichment`

### Data Models

`internal/model/` defines all data structures:
- `AnalysisResult`: Top-level output containing metadata, sections, imports, exports, strings, functions
- `BinaryMetadata`: File info, hashes, format, architecture
- `Capability`: Behavioral capability with ATT&CK/MBC mappings (from capa)
- `PackerDetection`: Packer/compiler/linker detection (from diec)
- `Enrichment`: Raw tool output storage
- Query commands filter and return subsets of cached AnalysisResult

### Output Formatters

`internal/output/` handles JSON and Markdown formatting. All commands default to Markdown (`-o md`), use `-o json` for JSON.

## Global Flags

- `-o, --output`: Output format (`json`, `md`) - default: `md`
- `-t, --timeout`: Analysis timeout - default: `2m0s`
- `-v, --verbose`: Verbose output

## Investigation Workflows

### Quick Triage
Fast initial assessment of a suspicious binary:
```bash
lcre query summary <binary>      # Overview with YARA matches and counts
lcre query yara <binary>         # Check YARA signature matches
lcre query iocs <binary>         # Extract IOCs (URLs, IPs, domains, paths)
```

### Malware Analysis
Deep analysis for confirmed or suspected malware:
```bash
lcre query summary <binary>                        # Initial risk assessment
lcre query functions <binary>                      # List all functions (triggers Ghidra)
lcre query decompile <binary> <suspicious_func>    # Examine suspicious functions
lcre query call-path <binary> main <target_func>   # Trace how malicious functions are reached
```

### IOC Extraction
Comprehensive IOC extraction for threat intelligence:
```bash
lcre query iocs <binary>                           # Extract IOCs from cached analysis
lcre query strings --pattern http <binary>         # Find URL-related strings
lcre query strings --pattern "C:\\" <binary>       # Find Windows file paths
lcre query imports --library ws2_32 <binary>       # Check for networking imports
lcre query imports --library wininet <binary>      # Check for HTTP/internet imports
```

### Packed Binary Analysis

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [refractionPOINT/lcre](https://github.com/refractionPOINT/lcre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
