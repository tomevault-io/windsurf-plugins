---
trigger: always_on
description: **Project:** MediaLyze
---

# AGENTS.md

**Project:** MediaLyze  
**Repository Type:** Open-source self-hosted media analysis tool  
**Primary Goal:** Analyze large video media collections with `ffprobe`, persist normalized technical metadata in SQLite, and expose performant inspection, statistics, and scan-management workflows through a FastAPI + React application.

---

# 1. Current State Snapshot

MediaLyze is no longer a greenfield v1 concept document. This file describes the **current `dev` branch implementation state** and should be treated as an engineering overview for agents working in this repository.

Current baseline:

* main branch: `main`
* branch basis: `dev`
* primary development branch: `dev`
* stack: **Python 3.12**, **FastAPI**, **SQLAlchemy**, **SQLite**, **React 19**, **Vite**, **TypeScript**, **i18next**, **APScheduler**, **watchdog**, **Electron**, **Docker**, **GHCR**

Current `dev` already includes unreleased additions beyond `v0.2.0`, including:

* path-browser filtering for placeholder directories such as `cdrom`, `floppy`, and `usb` when they are only container-exposed shadow directories
* broader HDR10+ detection from additional ffprobe side-data metadata variants

Important documentation rule:

* prefer the actual repository code and GitHub release metadata over `CHANGELOG.md` when they disagree
* `CHANGELOG.md` is currently incomplete on `dev` and does **not** fully reflect the already published `v0.2.0` release
* `main` is the primary stable / release branch, while `dev` is the primary ongoing development branch

---

# 2. Product Scope

MediaLyze is a **self-hosted technical media analyzer** for video collections.  
It focuses on file analysis, scan orchestration, metadata normalization, and library statistics.

## 2.1 Implemented Now

MediaLyze currently implements:

* library creation, update, rename, and deletion
* per-library dashboard visibility toggles that can exclude selected libraries from dashboard statistics and comparison panels
* safe directory browsing restricted to paths under `MEDIA_ROOT`
* manual, scheduled, and watchdog-based scanning
* full and incremental scans
* scan cancelation
* recent scan logs and detailed scan-job summaries
* deterministic change detection using path, file size, and modification time
* ffprobe-based normalization of media, stream, subtitle, and raw payload data
* internal and external subtitle detection
* configurable per-library quality profiles
* per-file quality score breakdowns
* per-library duplicate detection using filename signatures, exact file hashes, or both together
* structured metadata search, filtering, sorting, and pagination
* dashboard and per-library statistics
* dashboard and per-library metric-comparison panels with selectable X/Y dimensions and heatmap, scatter, or bar renderers where supported
* theme selection and feature flags
* English and German UI translations
* Docker-first deployment and GHCR image publishing
* native desktop packaging for Windows, macOS, and Linux with a local backend sidecar

## 2.2 Explicit Non-Goals

MediaLyze does **not** currently:

* play media
* scrape movie or TV metadata
* connect to external metadata APIs
* modify, rename, or transcode media files
* manage authentication internally

## 2.3 Backlog / Not Yet Implemented

Open or clearly future-facing work includes:

* improved broken-file reporting and diagnostics
* additional future analysis and recommendation workflows

These items should be treated as backlog, not current behavior.

---

# 3. Core Runtime Behavior

## 3.1 Libraries

Libraries represent directories below `MEDIA_ROOT` in server mode and absolute filesystem paths in desktop mode.

Each library currently stores:

* name
* absolute resolved path
* library type
* `scan_mode`
* `duplicate_detection_mode`
* `scan_config`
* `quality_profile`
* `show_on_dashboard`
* timestamps such as `created_at`, `updated_at`, and `last_scan_at`

Supported library types:

```text
movies
series
mixed
other
```

Important correction:

* the current code preserves the library type enum but does **not** implement special series-specific parsing that should be documented as an active feature

## 3.2 Path Browsing Safety

The server-mode UI path browser is constrained to `MEDIA_ROOT`.

Current behavior includes:

* rejecting paths outside `MEDIA_ROOT`
* skipping symlinks that resolve outside `MEDIA_ROOT`
* hiding placeholder container directories like `cdrom`, `floppy`, and `usb` when they are not real intended media targets
* keeping explicit mounted directories visible when they are valid browse targets
* desktop builds using a native OS folder picker plus absolute-path validation instead of the backend browse tree

## 3.3 Scan Modes

Libraries support three active scan modes:

```text
manual
scheduled
watch
```

Behavior:

* `manual`: scans run only when requested
* `scheduled`: APScheduler creates interval-based scan jobs
* `watch`: watchdog observers debounce filesystem events and queue scans
* desktop network paths fall back to `scheduled`; watch observers are only created for local desktop paths

## 3.4 Scan Types

The current API and runtime support:

```text
full
incremental
```

Current scan execution behavior:

1. traverse the library directory deterministically
2. apply ignore-pattern filtering during discovery

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frederikemmer/MediaLyze](https://github.com/frederikemmer/MediaLyze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
