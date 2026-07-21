---
trigger: always_on
description: This file provides guidance for AI coding agents working on the `osv.dev` repository. It outlines the project structure, setup commands, testing instructions, and coding standards.
---

# AGENTS.md

This file provides guidance for AI coding agents working on the `osv.dev` repository. It outlines the project structure, setup commands, testing instructions, and coding standards.

> [!IMPORTANT]
> **Keeping this file up to date**: If you (an AI agent) make any major architectural changes, introduce new services, or modify core workflows (like testing or linting), you **MUST** update this `AGENTS.md` file to reflect those changes.

## Project Overview
This repository (`google/osv.dev`) contains the backend services, database models, API, and website for the Open Source Vulnerabilities (OSV) database.

### Infrastructure & Storage
- **Cloud Platform**: The entire system runs on **Google Cloud Platform (GCP)**.
- **Database (Indexes)**: We currently use **Google Cloud Datastore** to store and query indexes.
- **Database (Full Records)**: Full vulnerability records are stored as **protocol buffers (protos) in Google Cloud Storage (GCS)**.
- *Future Architecture*: There are long-term plans to migrate the database backend to **PostgreSQL**, but this is not yet concrete. **Rule**: Any new Go code interacting with the database must be abstracted behind interfaces to facilitate this eventual migration. Much of this abstraction is already in place—refer to the shared domain interfaces defined under [`go/internal/models/`](go/internal/models/).

### Monorepo Structure
It is structured as a multi-language monorepo:
- `go/`: Go services and utilities (importers, exporter, internal libraries). **This is the primary target for active migrations from Python.**
- `osv/`: Core Python library containing models, repository helpers, and ecosystem-specific logic. *Note: Some parts are deprecated as we migrate logic to Go.*
- `gcp/`: GCP deployment configurations, Cloud Functions, API server, and workers. (Website frontend uses `pnpm` and Hugo).
- `vulnfeeds/`: Vulnerability feed utilities (*independent Go module*).
- `bindings/`: API bindings (*contains an independent Go module under `bindings/go`*).

### OSV Schema Reference
Vulnerabilities across the entire system conform to the **Open Source Vulnerability (OSV) schema**.
When AI agents need to understand the exact format, fields, and semantics of vulnerability records, refer to the local `osv-schema` submodule:
- **Full Specification**: [`osv/osv-schema/docs/schema.md`](osv/osv-schema/docs/schema.md)
- **Protobuf Definition**: [`osv/osv-schema/proto/vulnerability.proto`](osv/osv-schema/proto/vulnerability.proto)
- **JSON Schema**: [`osv/osv-schema/validation/schema.json`](osv/osv-schema/validation/schema.json)

---

## Datastore Schema & Entities
We use Google Cloud Datastore to store indices and metadata for fast querying. The primary source of truth for full vulnerability records is GCS (as protobufs), but Datastore holds crucial entities for the API and Website.

These models are defined in Python ([`osv/models.py`](osv/models.py)) and mirrored in Go ([`go/internal/database/datastore/models.go`](go/internal/database/datastore/models.go)).

### Key Entities

1. **`Vulnerability` (Kind: `Vulnerability`)**
   * **Purpose**: Serves as the main index for vulnerability metadata (source, modified time, aliases, relations).
   * **Fields**: Stores `source_id` (e.g., `source:path`), `modified` time, and relation lists (`alias_raw`, `related_raw`, `upstream_raw`).

2. **`AffectedVersions` (Kind: `AffectedVersions`)**
   * **Purpose**: Used for API matching when querying by package name and version.
   * **Fields**: Contains `ecosystem`, `name` (package name), `versions` (list of affected versions), and `events` (introduced/fixed ranges).
   * **Optimization**: Uses `coarse_min` and `coarse_max` for fast range-based filtering.

3. **`AffectedCommits` (Kind: `AffectedCommits`)**
   * **Purpose**: Used for API matching when querying by Git commit.
   * **Fields**: Maps a `bug_id` (vulnerability ID, note the legacy field name) to a list of affected Git commit hashes (stored as bytes).
   * **Schema Quirk**: The field for vulnerability ID is `bug_id` in Datastore but mapped to `VulnID` in Go.

4. **`ListedVulnerability` (Kind: `ListedVulnerability`)**
   * **Purpose**: Optimized specifically for the website's `/list` page.
   * **Fields**: Contains summary, ecosystems, packages, severities, and search indices.
   * **Rule**: This entity is **only** used by the website and should not be used for API matching logic.

---

## Setup Commands
The project uses `poetry` for Python dependency management, `pnpm` for website frontend, and Standard Go modules for Go.

- **Install Python Dependencies**:
  ```bash
  poetry install
  ```
- **Install Go Dependencies**:
  There are multiple Go modules in this monorepo. Run `go mod download` from within the respective directory (`go/`, `vulnfeeds/`, or `bindings/go/`) depending on what you are working on:
  ```bash
  cd go && go mod download
  ```
- **Install Website Dependencies** (for frontend development):
  ```bash
  cd gcp/website/frontend3 && pnpm install
  ```
- **Initialize Git Submodules**:
  ```bash
  git submodule update --init --recursive
  ```
- **Build Protos**:
  ```bash
  make build-protos
  ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/osv.dev](https://github.com/google/osv.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
