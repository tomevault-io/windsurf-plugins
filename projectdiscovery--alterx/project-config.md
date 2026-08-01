---
trigger: always_on
description: Generates all combinations of payloads across variables:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AlterX is a fast and customizable subdomain wordlist generator using DSL (Domain-Specific Language). This is a **Go port** that integrates pattern mining capabilities from [Regulator](https://github.com/cramppet/regulator) by @cramppet into the original [ProjectDiscovery alterx](https://github.com/projectdiscovery/alterx).

**Key Features:**
- Template-based subdomain generation using variables like `{{sub}}`, `{{suffix}}`, `{{word}}`
- Pattern mining mode that automatically discovers subdomain patterns from observed data
- Three operation modes: default (user patterns), discover (mined patterns), both (combined)
- ClusterBomb attack pattern for generating permutations

## Build & Development Commands

```bash
# Build the binary
make build

# Run tests
make test

# Run tests with coverage
make test-coverage

# Run linter (requires golangci-lint)
make lint

# Format code
make fmt

# Clean build artifacts
make clean

# Install to $GOPATH/bin
make install

# Build and run help
make run
```

**Single test execution:**
```bash
go test -v -run TestFunctionName ./path/to/package

# Run specific test at package root
go test -v -run TestMutator
go test -v -run TestInput

# Run with race detector
go test -race -v ./...
```

## Architecture

### Core Components

**1. Entry Point** (`cmd/alterx/main.go`)
- CLI argument parsing via `runner.ParseFlags()` using goflags library
- Mode selection logic (default/discover/both) passed to `alterx.Options`
- Pattern mining flow orchestration in `Mutator.Execute()` via goroutines
- Output writing with `getOutputWriter()` (file or stdout)
- Rules saving via `Mutator.SaveRules()` after execution completes

**2. Mutator Engine** (`mutator.go`, `algo.go`)
- `Mutator` struct: Core permutation generator with concurrent execution
- `Execute()` method: Runs default and/or mining modes in parallel goroutines
- `ClusterBomb` algorithm: Recursive Nth-order payload combination (cartesian product)
- `IndexMap`: Maintains deterministic ordering for payload iteration
- Template replacement using `fasttemplate` library with `{{var}}` syntax
- Deduplication via `dedupe.NewDedupe()` with configurable memory limits
- Smart optimization: Skips words already present in leftmost subdomain

**3. Input Processing** (`inputs.go`)
- `Input` struct: Parses domains into components (sub, suffix, tld, etld, etc.)
- Uses `publicsuffix` library to extract eTLD and root domain correctly
- Variable extraction: `{{sub}}`, `{{sub1}}`, `{{suffix}}`, `{{root}}`, `{{sld}}`, etc.
- Multi-level subdomain support (e.g., `cloud.api.example.com` → `sub=cloud`, `sub1=api`)
- `getNValidateRootDomain()`: Validates homogeneous domains for pattern mining

**4. Pattern Mining** (`internal/patternmining/`)
- **Three-phase discovery algorithm:**
  1. Edit distance clustering (no prefix enforcement)
  2. N-gram clustering (unigrams/bigrams)
  3. N-gram prefix clustering with edit distance refinement
- **Quality control:** Pattern threshold and quality ratio prevent over-generation
- **Regex generation:** Converts clusters to patterns with alternations `(a|b)` and optional groups `(...)?`
- **Number compression:** Optimizes `[0-9]` ranges automatically

**5. DFA Engine** (`internal/dank/dank.go`)
- Brzozowski's algorithm for DFA minimization
- Thompson NFA construction from regex
- Subset construction for NFA→DFA conversion
- Reverse DFA for minimization (determinize → reverse → determinize → reverse → determinize)
- Fixed-length string generation from automaton

### File Structure

```
cmd/alterx/main.go          # Entry point, mode selection, orchestration
internal/runner/
  ├── runner.go             # CLI flag definitions and parsing
  ├── config.go             # Version and config management
  └── banner.go             # Banner display
internal/patternmining/
  ├── patternmining.go      # Main mining algorithm (3 phases)
  ├── clustering.go         # Edit distance clustering logic
  └── regex.go              # Tokenization and regex generation
internal/dank/
  └── dank.go               # DFA-based pattern generation (Brzozowski)
mutator.go                  # Core Mutator with ClusterBomb algorithm
algo.go                     # ClusterBomb implementation and IndexMap
inputs.go                   # Domain parsing and variable extraction
replacer.go                 # Template variable replacement
config.go                   # Default patterns and payloads
util.go                     # Helper functions
```

## Key Concepts

### Variables System
Templates use variables extracted from input domains:
- `{{sub}}`: Leftmost subdomain part (e.g., `api` in `api.example.com`)
- `{{suffix}}`: Everything except leftmost part (e.g., `example.com`)
- `{{root}}`: eTLD+1 (e.g., `example.com`)
- `{{sld}}`: Second-level domain (e.g., `example`)
- `{{tld}}`: Top-level domain (e.g., `com`)
- `{{etld}}`: Extended TLD (e.g., `co.uk`)
- `{{subN}}`: Multi-level support where N is depth (e.g., `{{sub1}}`, `{{sub2}}`)

### ClusterBomb Algorithm
Generates all combinations of payloads across variables:
- Uses recursion with vector construction
- Maintains deterministic ordering via IndexMap

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectdiscovery/alterx](https://github.com/projectdiscovery/alterx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
