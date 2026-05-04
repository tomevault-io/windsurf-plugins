---
trigger: always_on
description: This document provides technical details about the `vercel-deploy-source-downloader` tool for AI agents and developers working with the codebase.
---

# AGENTS.md - Technical Documentation for AI Agents & Developers

This document provides technical details about the `vercel-deploy-source-downloader` tool for AI agents and developers working with the codebase.

## Architecture Overview

### Core Components

1. **Configuration Management**
   - `.env` file loader with priority system
   - CLI argument parser
   - Environment variable support
   - Priority: CLI > ENV > .env > defaults

2. **Vercel API Integration**
   - Deployments API (v6) - List and fetch deployment info
   - Deployments API (v13) - Get specific deployment details with auto-detection
   - Teams API (v2) - Resolve team scopes
   - File Tree API - Get directory structure (lazy-loaded per directory)
   - Files API (v7) - Download individual files (hash-based and path-based URLs)

3. **File Management**
   - Recursive directory traversal with lazy child loading
   - Smart caching (skip existing files)
   - Deployment-specific output folders (`out/{deploymentId}/source/`)
   - Resume support with pre-existing file count
   - Retry failed downloads (interactive + `--retry-failed` flag)

4. **Logging & Reporting**
   - Dual logging (console + file)
   - Append-mode log file with run separators (preserves history)
   - Verbose mode support
   - Statistics collection
   - Tree view visualization

5. **Progress Display**
   - Multi-line spinner with live counters (downloaded, skipped, failed)
   - TTY-aware (ANSI codes only when stdout is a terminal)
   - Spinner disabled in verbose mode (per-file log messages instead)

## File Structure

```
vercel-deploy-source-downloader/
├── src/
│   └── vercel-deploy-source-downloader.ts # Main script
├── docs/
│   ├── configuration.md                   # CLI args, env vars, interactive mode
│   ├── finding-deployment-id.md           # How to get deployment IDs
│   ├── advanced.md                        # Resume, retry, verbose, output structure
│   └── troubleshooting.md                 # Common errors and fixes
├── package.json                           # NPM package config
├── .env.example                           # Example environment config
├── README.md                              # User documentation
├── AGENTS.md                              # This file
├── CHANGELOG.md                           # Version history
├── PUBLISHING.md                          # NPM publishing guide
├── .gitignore                             # Git ignore rules
└── LICENSE                                # MIT license
```

## API Endpoints Used

### 1. List Deployments
```
GET https://api.vercel.com/v6/deployments?limit=100&teamId={teamId}
```
**Purpose:** Fetch available deployments, filter by project/state

### 2. Get Deployment Info
```
GET https://api.vercel.com/v13/deployments/{deploymentId}?teamId={teamId}
```
**Purpose:** Get specific deployment details. Tried with raw ID and `dpl_` prefix across personal account and all teams.

### 3. List Teams
```
GET https://api.vercel.com/v2/teams
```
**Purpose:** Resolve all team scopes for auto-detection.

### 4. File Tree
```
GET https://vercel.com/api/file-tree/{deploymentUrl}?base=src/{path}&teamId={teamId}
```
**Purpose:** Get source file tree structure. Directories are lazy-loaded — top-level returns immediate children only; subdirectories require separate requests with `base=src/{dirPath}`.

**Response:**
```typescript
Array<{
  name: string;
  type: "file" | "directory" | "lambda";
  link?: string;    // For files: download URL (hash-based or path-based)
  children?: FileNode[];  // For directories (may be absent — lazy-loaded)
}>
```

### 5. Download File

Two URL formats returned by the File Tree API:

**Hash-based** (older deployments):
```
https://vercel.com/api/v7/deployments/{deploymentId}/files/{hash}?teamId={teamId}
```

**Path-based** (newer deployments):
```
https://vercel.com/api/v7/deployments/{deploymentId}/files/get?path={filePath}
```

`resolveFileUrl(link)` normalizes both formats into a full URL with `teamId`.

**Response:** `{ data: string }` (base64 encoded). Some files (e.g. `.env*`) are blocked by Vercel with `{ error: { message: "Previewing this file is not supported." } }`.

## Key Functions

### `loadEnvFile()`
- Reads `.env` file from current directory
- Parses KEY=VALUE format
- Handles quoted values
- Only sets if not already in environment

### `parseArgs()`
- Loads .env first
- Parses CLI arguments (including `--retry-failed` flag)
- Returns configuration object with priority handling

### `getLatestDeployment()`
- Fetches deployment list
- Filters by project name (if specified)
- Filters to READY state only
- Sorts by creation time (newest first)
- Returns deployment details

### `resolveFileUrl(link: string): string | null`
- Normalizes file tree link into a full download URL
- Handles hash-based, full URL, and relative URL formats
- Appends `teamId` if missing

### `downloadFile(fileUrl: string): Promise<Buffer>`
- Downloads file via resolved URL
- Decodes base64 response
- Detects API error responses (e.g. blocked `.env*` files)
- Returns raw buffer

### `processNode(node, basePath, relativePath)`
- Recursive tree traversal
- Lazy-loads directory children (stores back into tree for retry)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [numanaral/vercel-deploy-source-downloader](https://github.com/numanaral/vercel-deploy-source-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
