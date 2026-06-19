---
trigger: always_on
description: Welcome, Agent! This guide contains crucial context, structural guidelines, and best practices for working in the `okf-skills` repository. Follow these principles to maintain high-quality, spec-compliant, and portable implementations.
---

# Open Knowledge Format (OKF) Skills — Developer Agent Guide

Welcome, Agent! This guide contains crucial context, structural guidelines, and best practices for working in the `okf-skills` repository. Follow these principles to maintain high-quality, spec-compliant, and portable implementations.

---

## 1. Repository Overview & Architecture

This repository is a central collection of skills for producing, consuming, and authoring Open Knowledge Format (OKF) bundles — standalone CLI connectors, instructions-only guidance skills (read, enrich, and author), and a generic MCP server. It is organized as a Go workspace containing multiple modules:

```
okf-skills/
├── AGENTS.md                      # This guide
├── README.md                      # General user-facing overview
├── LICENSE                        # Apache License 2.0
├── go.work                        # Go workspace defining monorepo modules
├── Makefile                       # Build, test, install shortcuts
├── install.sh                      # Build and install all skills to a directory
├── skills.sh.json                 # skills.sh registry manifest (groups skills for discovery)
├── okf-go/                        # Shared Go library (YAML/MD serialization, ignore/metadata helpers)
│   ├── okf.go                     # Core types: Frontmatter, ConceptDoc, helpers
│   ├── okf_test.go                # Unit tests
│   └── okf-SPEC.md               # Full OKF specification document
├── okf-mcp/                       # Generic MCP server — the host that exposes skills (not a skill)
├── skills/                        # Standalone Go-based CLI skills
│   ├── okf-sqlite/                # SQLite connector (CGO-free)
│   ├── okf-mysql/                 # MySQL connector
│   ├── okf-postgresql/            # PostgreSQL connector
│   ├── okf-bigquery/              # GCP BigQuery connector
│   ├── okf-fs/                    # Local filesystem connector
│   ├── okf-git/                   # Git repository connector
│   ├── okf-enrich/                # Enrichment guidance skill (Instructions-only)
│   ├── okf-reader/                # Ingestion guidance skill (Instructions-only)
│   ├── okf-producer-generator/    # Producer-authoring guidance skill (Instructions-only)
│   └── okf-viz/                   # Bundle visualizer — renders OKF bundles to interactive HTML
└── tests/                         # Central integration testing directory
    ├── docker-compose.yml         # MySQL & PostgreSQL containers
    ├── helpers_test.go            # Shared test utilities (getBinaryPath, isPortOpen, etc.)
    ├── db_integration_test.go     # SQLite, MySQL, PostgreSQL integration tests
    ├── fs_integration_test.go     # Filesystem integration tests
    ├── git_integration_test.go    # Git integration tests
    ├── mysql/
    │   └── init_mysql.sql         # Sample MySQL schema with comments
    ├── postgres/
    │   └── init_postgres.sql      # Sample PostgreSQL schema with comments
    └── testdata/                   # Test fixtures & sample data
```

---

## 2. Shared Library (`okf-go`) Guidelines

All core OKF schemas and parsing helper functions live under `okf-go/`.
- **Do Not Duplicate Structs**: The `Frontmatter` and `ConceptDoc` structs must not be defined in individual skills. Import `github.com/xSAVIKx/okf-skills/okf-go` instead.
- **Spec Compliance**: OKF concepts are Markdown files with YAML frontmatter.
  - Subdirectory `index.md` files must contain **no frontmatter**.
  - The bundle-root `index.md` is the only index permitted to contain frontmatter, and it should only declare `okf_version: "0.1"` (omit `type`, `title`, and `description` from the YAML block; place them directly inside the Markdown body).
- **Line Ending Compatibility**: `ReadConceptDoc` split operations must handle both Unix LF (`\n`) and Windows CRLF (`\r\n`) markers for frontmatter boundaries.
- **Ignore & Metadata Helpers**: Use the shared `IgnoreMatcher` helper to load `.okfignore` wildcard matchers, and `ReadFolderMetadata`/`WriteFolderMetadata` to serialize/deserialize path-to-description mapping inside `.okf-metadata.yaml`.
- **Unit Testing**: Maintain robust tests in `okf_test.go` and run `go test -v ./...` inside `okf-go/` after making changes.

---

## 3. Skills Development & Best Practices

The connector skills compile to standalone Go CLI binaries, each exposing three subcommands (the `okf-viz` visualizer is a consumer binary exposing `render`/`schema`; the guidance skills are instructions-only, no binary):
1. `produce`: Extract database schema comments, local filesystem folder structures, or git repository commit history into an OKF bundle. The four SQL connectors (`okf-sqlite`, `okf-mysql`, `okf-postgresql`, `okf-bigquery`) also support `--sample` and `--profile` flags.
2. `ingest`: Read an OKF bundle, validate assets, and optionally synchronize comments/descriptions back to the source database or `.okf-metadata.yaml` using the `-sync` flag.
3. `schema`: Emit a JSON description of the skill's commands, flags, and parameters (used by `okf-mcp` for tool discovery).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xSAVIKx/okf-skills](https://github.com/xSAVIKx/okf-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
