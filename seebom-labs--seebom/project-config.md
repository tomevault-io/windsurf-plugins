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
| `api-gateway` | Deployment | Stateless REST API (16 endpoints) |
| `cve-refresher` | K8s CronJob (daily) | Checks all known PURLs for newly disclosed CVEs without re-scanning SBOMs |

Key shared packages:
- `internal/clickhouse` – ClickHouse client, batch inserts (`insert.go`), queue operations (`queue.go`), and all query logic split across `queries.go`, `queries_search.go`, `queries_refresh.go`, `queries_github_cache.go`
- `internal/config` – Environment-based configuration loader (`config.Load()` reads env vars with sensible defaults)
- `internal/repo` – Directory scanner with SHA256 hashing and file type classification (`.spdx.json` → sbom, `.openvex.json` → vex)
- `internal/osvutil` – Shared OSV helpers (ClassifySeverity, ExtractFixedVersion, ExtractAffectedVersions)
- `internal/license` – License compliance + externalized policy + exceptions with prefix-matching
- `internal/osv` – OSV API client with rate limiting and exponential backoff
- `internal/s3` – S3-compatible bucket client (AWS S3, MinIO, GCS) for streaming SBOM ingestion from multiple buckets
- `internal/github` – GitHub API client for resolving unknown licenses from PURL (rate-limited, cached). Includes 50+ well-known Go module→GitHub repo mappings (`golang.org/x/*`, `gopkg.in/*`, `go.uber.org/*`, `k8s.io/*`, `oras.land/*`, `dario.cat/*`, etc.), fallback to the dedicated `/repos/{owner}/{repo}/license` endpoint, and static license overrides for repos where GitHub misdetects the license.
- `internal/spdx` – SPDX JSON streaming parser. Supports both plain SPDX documents and **in-toto attestation envelopes** where the SPDX content is wrapped inside the `predicate` field (common with Syft/BuildKit-generated container SBOMs).
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

**Deployment Strategy:** We use a hybrid approach. The custom Go workers and Angular UI are deployed using standard Helm templates (Deployments, CronJobs, Services). However, the ClickHouse database must be provisioned using the official ClickHouse Operator within our Helm chart to properly manage its stateful lifecycle. Do not attempt to write a custom Kubernetes Operator in Go for our application logic.

**Config-Driven Governance:** License policy (`license-policy.json`) and license exceptions (`license-exceptions.json`) are externalized as config files – mounted via Docker Compose volumes locally and Kubernetes ConfigMaps in production. The frontend is public, so no write APIs for exceptions exist. Changes require config file updates + re-ingest.

**CVE Refresh Strategy:** New CVEs are discovered via a lightweight daily CronJob (`cve-refresher`) that queries all unique PURLs (~20k) against the OSV API in 1000-PURL batch chunks, deduplicates against existing vulnerabilities, and inserts new findings. This avoids expensive full re-scans of all SBOMs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seebom-labs/seebom](https://github.com/seebom-labs/seebom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
