---
trigger: always_on
description: PRs should be created against `ironicbadger/quicksync_calc` (NOT `cptmorgan-rh`).
---

# QuickSync Benchmark - Interactive Website & Database Plan

## GitHub

PRs should be created against `ironicbadger/quicksync_calc` (NOT `cptmorgan-rh`).

## Python

Use `uv` for Python dependency management. Use `uv run --with <package>` to run Python scripts with dependencies without installing them globally.

## Project Overview

Transform the QuickSync Video benchmarking tool from GitHub Gist-based data collection into a modern interactive web application with proper database storage, API submission, and rich visualization.

## Current Architecture

```mermaid
flowchart LR
    subgraph User
        A[Run benchmark.sh]
    end

    subgraph Manual
        B[Copy results]
        C[Paste to GitHub Gist]
    end

    subgraph Automation
        D[GitHub Actions weekly]
        E[analysis.py]
        F[plot.png]
    end

    A --> B --> C
    D --> E --> F
    C -.-> E
```

**Problems:**
- Manual copy/paste workflow
- No filtering or interactivity
- Gist comments are hard to query
- Intel's new CPU naming breaks generation parsing

## Proposed Architecture

```mermaid
flowchart TB
    subgraph Benchmark["Benchmark Execution"]
        A[quicksync-benchmark.sh]
        B[concurrency-benchmark.sh]
    end

    subgraph API["Cloudflare Workers API"]
        C[POST /api/submit]
        D[POST /api/submit-concurrency]
        E[GET /api/results]
        F[GET /api/stats]
    end

    subgraph Storage["Cloudflare R2"]
        G[(benchmarks.json)]
    end

    subgraph Website["Vite + React Static Site"]
        J[Interactive Charts]
        K[Filter Controls]
        L[Results Table]
    end

    subgraph Migration
        M[migrate-gist-data.py]
        N[GitHub Gist API]
    end

    A -->|curl POST| C
    B -->|curl POST| D
    C --> G
    D --> G
    E --> G
    F --> G

    J --> E
    J --> F
    K --> E
    L --> E

    N --> M --> G
```

## Technology Stack

| Component | Technology | Why |
|-----------|------------|-----|
| Storage | Cloudflare R2 | Free tier, global edge, simple JSON storage |
| API | Cloudflare Workers | Free tier, global edge, native R2 support |
| Frontend | React 19 + Vite 7 + Chart.js | Fast SPA, predictable state, no imperative DOM |
| Hosting | Cloudflare Pages | Free, global CDN, custom domain support |

### Storage: Cloudflare R2

Using Cloudflare R2 with a single JSON file (`benchmarks.json`):

**Why R2:**
- Zero infrastructure to manage
- Global edge distribution
- Simple JSON file format (easy to backup/restore)
- Versioned backups on each write
- No query language needed - just read/write JSON

## Data Flow

```mermaid
sequenceDiagram
    participant User
    participant Script as benchmark.sh
    participant API as Cloudflare Worker
    participant R2 as R2 Storage
    participant Web as Website

    User->>Script: Run benchmark
    Script->>Script: Execute FFmpeg tests
    Script->>Script: Collect metrics

    alt Auto-submit enabled
        Script->>API: POST /api/submit (pipe-delimited)
        API->>API: Parse & validate
        API->>API: Extract CPU architecture
        API->>API: Compute fps_per_watt
        API->>API: Generate result_hash
        API->>R2: Read/Write JSON (deduplicated)
        R2-->>API: Success
        API-->>Script: {inserted: 5}
    else Manual
        Script->>User: Display results table
        User->>User: Copy to Gist (legacy)
    end

    User->>Web: Visit quicksync.ktz.me
    Web->>API: GET /api/results?filters
    API->>R2: Read JSON
    R2-->>API: Results
    API-->>Web: JSON response
    Web->>Web: Render interactive charts
```

## Data Schema

The `benchmarks.json` file stored in R2 contains all data in a single JSON structure:

```typescript
interface BenchmarkData {
  version: number;
  lastUpdated: string;
  meta: {
    totalResults: number;
    uniqueCpus: number;
    architecturesCount: number;
    uniqueTests: number;
  };
  architectures: CpuArchitecture[];
  results: BenchmarkResult[];
  concurrencyResults: ConcurrencyResult[];
  cpuFeatures: Record<string, CpuFeatures>;
}
```

## Intel CPU Architecture Timeline

Intel's naming scheme is now chaotic. We use a lookup table to map CPU strings to architectures.

```mermaid
timeline
    title Intel Quick Sync Video Evolution

    2011 : Sandy Bridge (2nd)
         : H.264 decode only

    2012 : Ivy Bridge (3rd)
         : H.264 encode

    2013 : Haswell (4th)
         : H.264 encode

    2015 : Skylake (6th)
         : HEVC decode

    2017 : Kaby Lake (7th)
         : HEVC 8-bit encode

    2018 : Coffee Lake (8-9th)
         : HEVC 8-bit encode

    2020 : Tiger Lake (11th)
         : HEVC 10-bit + VP9 encode

    2021 : Alder Lake (12th)
         : Full codec support (no AV1 encode)

    2022 : Arc Alchemist (dGPU)
         : AV1 encode!

    2023 : Meteor Lake (Ultra S1)
         : AV1 encode

    2024 : Arrow Lake (Ultra S2)
         : AV1 encode
```

## Hardware Codec Encoding Support

Source: [Intel Quick Sync Video - Wikipedia](https://en.wikipedia.org/wiki/Intel_Quick_Sync_Video)

| Architecture | Gen | Release | H.264 | HEVC 8-bit | HEVC 10-bit | VP9 | AV1 |
|--------------|-----|---------|-------|------------|-------------|-----|-----|
| Sandy Bridge | 2nd | 2011 | decode | - | - | - | - |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironicbadger/quicksync_calc](https://github.com/ironicbadger/quicksync_calc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
