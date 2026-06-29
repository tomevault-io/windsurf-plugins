---
trigger: always_on
description: This document provides guidance for AI agents working on arkade, specifically for reviewing and adding new CLI tools to the `arkade get` command.
---

# AGENTS.md - Guide for AI Agents Contributing new downloads/apps/CLIs to Arkade

This document provides guidance for AI agents working on arkade, specifically for reviewing and adding new CLI tools to the `arkade get` command.

For system-level packages (usually Linux-only), see the `arkade system install` - usually for tools like Node.js, Go, Prometheus. Sometimes uses Go templates for tool definitions, often just uses its own code in the command.
For binaries distributed via OCI images, see `arkade oci install`.

## Types of Arkade Commands

Arkade provides several types of installers:

- **`arkade get`** - CLI tools usually to be placed at `/usr/local/bin/` or `$HOME/.arkade/bin/`. These are standalone binaries that can be downloaded and executed directly.

- **`arkade system install`** - Linux-only system-level tools like Node.js, Go, Prometheus. These require additional installation steps or system configuration.

- **`arkade oci install`** - Fetches binaries out of OCI images. Ideal for projects that use private repositories like slicer/actuated/k3sup-pro.

- **`arkade install`** - Kubernetes Helm charts or manifests for add-ons like OpenFaaS CE, Istio, PostgreSQL. These deploy software to Kubernetes clusters.

**This guide focuses on `arkade get`** - adding CLI tools that provide static binaries for download.

## 1. How to Add a New CLI (Tool) to Arkade

### What Can Be Added

**Only tools with static binaries** can be added to arkade. The tool must provide pre-compiled binaries for download.

**Cannot be added:**
- Python-based tools (e.g., `aws-cli`, `azure-cli`) - require Python runtime
- Node.js-based tools without static binaries - require Node.js runtime
- Tools that require installation scripts or package managers
- Tools that need runtime dependencies beyond the binary itself

### Prerequisites

1. Fork and create a branch: `git checkout -b add-TOOL_NAME`

### Step 1: Check GitHub Releases

**CRITICAL**: Before writing code, check the latest stable release on GitHub to see what OS/architecture combinations are available.

1. Run a `curl -i -X HEAD https://github.com/OWNER/REPO/releases/latest` (adds `/latest` to go directly to the latest release) - change OWNER and REPO accordingly. The `location` header in the response will show the actual latest version tag without using up an API request. To obtain the `location` header, you must not use the `-L` (follow redirects) flag.
2. Examine ALL download URLs in the "Assets" section, you can obtain this via HTML, again to avoid consuming API requests: `https://github.com/OWNER/REPO/releases/expanded_assets/VERSION` - replace VERSION with the actual version tag from 1. and the OWNER/REPO accordingly. This returns HTML, you can grep it efficiently for anchor tags.
3. Note available combinations:
   - Linux amd64 (x86_64)
   - Linux arm64 (aarch64)
   - Darwin amd64 (x86_64)
   - Darwin arm64
   - Windows amd64 (x86_64)

**Important**: Match the exact naming used by the upstream project (`amd64` vs `x86_64`, `arm64` vs `aarch64`).

### Step 2: Add Tool Definition

Edit `pkg/get/tools.go` and add a new `Tool` entry. **Reference existing examples** like `faas-cli` (lines 27-50) for the structure.

**Key points:**
- Use `BinaryTemplate` for GitHub releases (simpler)
- Use `URLTemplate` for custom URLs or non-GitHub sources
- Supported archive formats: `.tar.gz`, `.zip` (`.tar.xz` is NOT supported)
- Template variables: `.OS`, `.Arch`, `.Name`, `.Version`, `.VersionNumber`, `.Repo`, `.Owner`
- Windows detection: `HasPrefix .OS "ming"`
- **CRITICAL**: If a binary is missing for a specific OS/arch (e.g., Windows amd64), the template must still generate a URL that results in a 404 error, NOT download the wrong binary (e.g., don't download Linux binary when Windows was requested)

#### Archive tools: when the binary name inside the archive differs from the tool name

When a tool is distributed as an archive (`.tar.gz`, `.tgz`, `.zip`) and the **binary inside the archive** has a platform-specific name (e.g., `mytool-darwin-arm64` rather than just `mytool`), you **must** use both `URLTemplate` and `BinaryTemplate` together:

- **`URLTemplate`** — the full download URL including the archive extension (e.g., `https://github.com/.../mytool-darwin-arm64.tgz`)
- **`BinaryTemplate`** — the name of the **binary inside the archive**, without the archive extension (e.g., `mytool-darwin-arm64`)

**Do NOT** put the archive filename (with `.tgz`/`.tar.gz`/`.zip` extension) in `BinaryTemplate` alone. The `decompress()` function in `pkg/get/download.go` uses `BinaryTemplate` to locate the extracted binary. If `BinaryTemplate` contains an archive extension, decompress falls back to `tool.Name` which will be wrong when the inner binary has a platform suffix.

**Reference example**: `inletsctl` in `pkg/get/tools.go` — uses `URLTemplate` for the download URL and `BinaryTemplate` for the inner binary name.

**When `BinaryTemplate` alone is safe**: Only when the tool is a **plain binary** (not an archive). In that case `BinaryTemplate` is the release asset filename, and the downloaded file is used directly without decompression.

### Step 3: Write Unit Tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexellis/arkade](https://github.com/alexellis/arkade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
