---
trigger: always_on
description: You are an expert Senior Software Engineer and Architect specializing in Go, Angular, Kubernetes, and high-performance analytical databases (ClickHouse).
---

# Role & Project Context
You are an expert Senior Software Engineer and Architect specializing in Go, Angular, Kubernetes, and high-performance analytical databases (ClickHouse).

We are building SeeBOM: a standalone, Kubernetes-native Software Bill of Materials (SBOM) visualization and governance platform. It autonomously ingests massive amounts of SPDX JSON files from the CNCF ecosystem (by default from S3-compatible buckets, with local filesystem as alternative), stores them for infinite historical retention, cross-references vulnerabilities via the OSV API, checks license compliance natively with externalized policy and exception files, supports VEX (Vulnerability Exploitability eXchange) via OpenVEX, and displays the results in a high-performance UI.

# Architecture Overview
The platform consists of **4 Go binaries**, an **Angular UI**, and a **ClickHouse** database:

| Binary | Type | Purpose |
|--------|------|---------|
| `ingestion-watcher` | K8s CronJob | Scans SBOM/VEX directory, hash-dedup, enqueues jobs |
| `parsing-worker` | Deployment (N replicas) | Processes SBOMs (SPDX→ClickHouse), VEX files, OSV lookups, license checks |
| `api-gateway` | Deployment | Stateless REST API (24 endpoints) |
| `cve-refresher` | K8s CronJob (daily) | Checks all known PURLs for newly disclosed CVEs without re-scanning SBOMs |

Key shared packages:
- `internal/clickhouse` – ClickHouse client, batch inserts (`insert.go`), queue operations (`queue.go`), and all query logic split across `queries.go`, `queries_projects.go`, `queries_search.go`, `queries_refresh.go`, `queries_github_cache.go`
- `internal/config` – Environment-based configuration loader (`config.Load()` reads env vars with sensible defaults)
- `internal/repo` – Directory scanner with SHA256 hashing and file type classification. Accepts any `.json` file (format auto-detected by parser), with configurable ignore prefix (`SBOM_IGNORE_PREFIX`, default `_`) to skip demo/example files. VEX files detected via `*.openvex.json` / `*.vex.json` suffix.
- `internal/osvutil` – Shared OSV helpers (ClassifySeverity, ExtractFixedVersion, ExtractAffectedVersions)
- `internal/license` – License compliance + externalized policy + exceptions with prefix-matching
- `internal/osv` – OSV API client with rate limiting and exponential backoff
- `internal/s3` – S3-compatible bucket client (AWS S3, MinIO, GCS) for streaming SBOM ingestion from multiple buckets. Supports per-bucket `cluster` assignment for multi-cluster differentiation from a single instance.
- `internal/github` – GitHub API client for resolving unknown licenses from PURL (rate-limited, cached). Includes 50+ well-known Go module→GitHub repo mappings (`golang.org/x/*`, `gopkg.in/*`, `go.uber.org/*`, `k8s.io/*`, `oras.land/*`, `dario.cat/*`, etc.), fallback to the dedicated `/repos/{owner}/{repo}/license` endpoint, and static license overrides for repos where GitHub misdetects the license.
- `internal/spdx` – SPDX JSON streaming parser. Supports both plain SPDX documents and **in-toto attestation envelopes** where the SPDX content is wrapped inside the `predicate` field (common with Syft/BuildKit-generated container SBOMs).
- `internal/cyclonedx` – CycloneDX JSON parser. Maps components, licenses, and dependencies to the shared model.
- `internal/sbom` – Multi-format SBOM dispatch layer. Auto-detects format (SPDX, CycloneDX, in-toto) and routes to the appropriate parser. Supports opt-in protobom backend via `USE_PROTOBOM=true`.
- `internal/protobomparser` – Parser backend using [protobom](https://github.com/protobom/protobom) for maximum format coverage (SPDX 2.3 + CycloneDX 1.0–1.7). Opt-in alternative to built-in parsers.
- `internal/vex` – OpenVEX parser with URL normalization

Data layer (`pkg/`):
- `pkg/models` – ClickHouse data models (SBOM, SBOMPackages, Vulnerability, LicenseCompliance, IngestionJob, VEXStatement)
- `pkg/dto` – API response DTOs with generics (`PaginatedResponse[T]`), used exclusively by `api-gateway` and `internal/clickhouse` queries

# Tech Stack
- **Backend & Workers:** Go (Golang)
- **Database:** ClickHouse (managed via the official ClickHouse Kubernetes Operator)
- **Frontend:** Angular (TypeScript, standalone components, OnPush change detection)
- **Infrastructure:** Kubernetes (Standard Helm Chart, 19 templates)
- **Container Registry:** GitHub Container Registry (ghcr.io/seebom-labs/seebom/*)
- **Go Module Path:** `github.com/seebom-labs/seebom/backend`

# Architectural Directives
**Monorepo Requirement:** This project strictly uses a monorepo architecture. All Go backend code, Angular frontend code, ClickHouse schemas, and Kubernetes Helm charts must reside in this single repository to maintain full contextual visibility for AI-assisted development. Do not suggest splitting this into a polyrepo.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seebom-labs/BOMHort](https://github.com/seebom-labs/BOMHort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
