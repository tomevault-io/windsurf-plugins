---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Skills Reference

Detailed guidance is organized into skills in `.claude/skills/`. Claude will auto-invoke these based on context:

| Skill                  | Triggers                                    | Purpose                                    |
|------------------------|---------------------------------------------|--------------------------------------------|
| **benchmark-docs**     | "benchmark", "update benchmarks"            | Platform-specific benchmark documentation  |
| **markdown-tables**    | "format table", "align table"               | Fixed-width table formatting               |
| **simd-optimization**  | "SIMD", "AVX", "NEON", "vectorization"      | SIMD patterns and learnings                |
| **bit-optimization**   | "rank", "select", "popcount", "lookup table"| Bit-level optimization patterns            |
| **json-semi-indexing** | "JSON index", "semi-index", "cursor"        | JSON parsing implementation details        |
| **yaml-semi-indexing** | "YAML index", "YAML parser", "sequence item"| YAML parsing and debugging patterns        |
| **testing**            | "test", "assert", "coverage", "regression"  | Test quality patterns and anti-patterns    |
| **commit-msg**         | "commit message", "amend commit"            | Conventional commits format                |
| **knowledge-map**      | "knowledge map", "wiki", "update wiki"      | Maintain docs/ knowledge wiki              |
| **skill-writing**      | "create skill", "SKILL.md", "write skill"   | Best practices for writing Claude skills   |

## Knowledge Wiki

A structured knowledge base for this codebase lives in `docs/`. Start at [docs/index.md](docs/index.md) for a concept-oriented map of how the data structures, algorithms, SIMD implementations, and benchmarks relate to each other. The wiki pages cross-link to existing architecture docs, parsing docs, source files, and academic papers.

## AI Scratch Directory

Use `.ai/scratch/` for temporary files (git-ignored):

```bash
mkdir -p .ai/scratch
```

## Project Overview

Succinctly is a high-performance Rust library implementing succinct data structures with fast rank and select operations, optimized for both x86_64 (POPCNT) and ARM (NEON) architectures.

### Semi-Indexing Architecture

Succinctly uses **semi-indexing** rather than traditional DOM parsing. Instead of building a complete in-memory tree, it creates a lightweight structural index (~3-6% overhead) and extracts values lazily. This enables:

- **17-46x less memory** than DOM parsers
- **Fast queries** because only accessed values are materialized
- **Streaming output** without intermediate allocations

**Trade-off**: Semi-indexing performs minimal validation compared to full parsers (jq, yq). The benchmark comparisons to jq/yq reflect both architectural advantages and reduced validation work. See [docs/architecture/semi-indexing.md](docs/architecture/semi-indexing.md) for details.

## Common Commands

### Building and Testing

```bash
cargo build                              # Standard build
cargo build --features simd              # With SIMD popcount
cargo test                               # Run tests
cargo test --features large-tests        # 1GB bitvector tests
cargo bench                              # Run benchmarks
```

### CLI Tool

```bash
cargo build --release --features cli

# Install short aliases (sjq, syq, sjq-locate, syq-locate)
./target/release/succinctly install-aliases          # symlinks next to binary
./target/release/succinctly install-aliases --dir ~/bin  # or specify directory

# JSON operations (sjq is an alias for succinctly jq)
sjq '.users[].name' input.json
sjq -r '.users[] | [.name, .age] | @csv' input.json
sjq -r '.users[] | [.name, .age] | @dsv("|")' input.json
sjq --validate '.users[]' input.json  # Strict RFC 8259 validation before processing
sjq-locate input.json --offset 42
sjq-locate input.json --line 5 --column 10

# JSON validation (RFC 8259 strict)
./target/release/succinctly json validate input.json
./target/release/succinctly json validate --quiet input.json  # Exit code only

# YAML operations (syq is an alias for succinctly yq)
syq '.users[].name' config.yaml
syq -o json '.' config.yaml         # Output as JSON
syq '.spec.containers[]' k8s.yaml
syq --doc 0 '.' multi-doc.yaml      # First document only
syq-locate config.yaml --offset 42
syq-locate config.yaml --line 5 --column 10

# DSV/CSV operations
sjq --input-dsv ',' '.[] | select(.[0] == "Alice")' data.csv
./target/release/succinctly dsv generate users 1000 -o users.csv

# Data generation
./target/release/succinctly json generate 10mb -o benchmark.json

# Benchmarks (requires: cargo build --release --features bench-runner)
./target/release/succinctly bench run jq_bench
./target/release/succinctly bench run yq_bench
./target/release/succinctly bench run yq_bench --queries all  # M2 streaming comparison (memory collected by default)
./target/release/succinctly bench run dsv_bench
```

### Multi-call Aliases


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rust-works/succinctly](https://github.com/rust-works/succinctly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
