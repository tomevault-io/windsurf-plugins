---
trigger: always_on
description: Manages portable CLI tool binaries with automated updates from upstream GitHub/GitLab/Kubernetes releases. Each tool is defined by a JSON file in `packages/`, and `scripts/update.sh` handles downloading, verifying, extracting, and installing binaries for both x86_64 and arm64 architectures. CI creates PRs for package updates; merging to master triggers a release via GitHub Actions.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Manages portable CLI tool binaries with automated updates from upstream GitHub/GitLab/Kubernetes releases. Each tool is defined by a JSON file in `packages/`, and `scripts/update.sh` handles downloading, verifying, extracting, and installing binaries for both x86_64 and arm64 architectures. CI creates PRs for package updates; merging to master triggers a release via GitHub Actions.

## Quick Reference

```bash
# Update all packages to latest upstream versions (local dev)
./scripts/update.sh

# Update specific packages
./scripts/update.sh nvim lazygit gh

# Update to a custom output directory
DOT_BIN_DIR=/tmp/staging/bin ./scripts/update.sh

# Override release channel at runtime (stable or unstable)
CHANNEL=unstable ./scripts/update.sh nvim

# Verify a binary after install
file bin/x86_64/<name>    # should show "ELF 64-bit LSB ... x86-64"
file bin/arm64/<name>     # should show "ELF 64-bit LSB ... ARM aarch64"

# List release assets for a GitHub repo (useful when adding packages)
gh release view --repo <owner/repo> --json assets --jq '.assets[].name'

# Check current installed versions
cat versions.json
```

## Package JSON Format

Each package is defined in `packages/<name>.json`. Fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Package identifier, must match filename |
| `repo` | string | yes* | GitHub `owner/repo` (*omit for GitLab/Kubernetes sources) |
| `source` | string | no | `"github"` (default), `"gitlab"`, or `"kubernetes"` |
| `gitlab_project` | string | no | URL-encoded GitLab project path (required when `source` is `"gitlab"`) |
| `tag_prefix` | string | no | Prefix stripped from git tag to get version (e.g. `"v"`, `"cli-v"`, `"jq-"`, `""`) |
| `channel` | string | no | `"stable"` (default) or `"unstable"` -- stable uses latest non-pre-release; unstable includes pre-releases |
| `format` | string | yes | `"tarball"`, `"zip"`, or `"binary"` |
| `output_binaries` | string[] | yes | Binary names placed in `bin/{arch}/` |
| `checksum.asset` | string | no | Checksum filename in the release (supports `{version}` placeholder) |
| `checksum.algorithm` | string | no | `"sha256"` or `"sha512"` |
| `architectures` | object | yes | Per-arch download and extract config (keys: `x86_64`, `arm64`) |
| `architectures.{arch}.asset_pattern` | string | yes | Download filename (supports `{version}` placeholder) |
| `architectures.{arch}.extract_path` | string or string[] | no | Path inside archive; use array for multi-binary packages; supports wildcards (e.g. `*/bin/gh`) |
| `architectures.{arch}.checksum_asset` | string | no | Per-arch checksum file, overrides `checksum.asset` (for per-asset checksums) |

### Examples

Simple tarball with checksum (`lazygit`):
```json
{
  "name": "lazygit",
  "repo": "jesseduffield/lazygit",
  "tag_prefix": "v",
  "format": "tarball",
  "output_binaries": ["lazygit"],
  "checksum": { "asset": "checksums.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "lazygit_{version}_linux_x86_64.tar.gz", "extract_path": "lazygit" },
    "arm64": { "asset_pattern": "lazygit_{version}_linux_arm64.tar.gz", "extract_path": "lazygit" }
  }
}
```

Standalone binary, no archive (`jq`):
```json
{
  "name": "jq",
  "repo": "jqlang/jq",
  "tag_prefix": "jq-",
  "format": "binary",
  "output_binaries": ["jq"],
  "checksum": { "asset": "sha256sum.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "jq-linux-amd64" },
    "arm64": { "asset_pattern": "jq-linux-arm64" }
  }
}
```

Multi-binary with per-asset checksums (`uv`):
```json
{
  "name": "uv",
  "repo": "astral-sh/uv",
  "tag_prefix": "",
  "format": "tarball",
  "output_binaries": ["uv", "uvx"],
  "checksum": { "algorithm": "sha256" },
  "architectures": {
    "x86_64": {
      "asset_pattern": "uv-x86_64-unknown-linux-gnu.tar.gz",
      "extract_path": ["uv-x86_64-unknown-linux-gnu/uv", "uv-x86_64-unknown-linux-gnu/uvx"],
      "checksum_asset": "uv-x86_64-unknown-linux-gnu.tar.gz.sha256"
    },
    "arm64": {
      "asset_pattern": "uv-aarch64-unknown-linux-gnu.tar.gz",
      "extract_path": ["uv-aarch64-unknown-linux-gnu/uv", "uv-aarch64-unknown-linux-gnu/uvx"],
      "checksum_asset": "uv-aarch64-unknown-linux-gnu.tar.gz.sha256"
    }
  }
}
```

GitLab source (`glab`):
```json
{
  "name": "glab",
  "source": "gitlab",
  "gitlab_project": "gitlab-org%2Fcli",
  "tag_prefix": "v",
  "format": "tarball",
  "output_binaries": ["glab"],
  "checksum": { "asset": "checksums.txt", "algorithm": "sha256" },
  "architectures": {
    "x86_64": { "asset_pattern": "glab_{version}_linux_amd64.tar.gz", "extract_path": "bin/glab" },
    "arm64": { "asset_pattern": "glab_{version}_linux_arm64.tar.gz", "extract_path": "bin/glab" }
  }
}
```

Kubernetes source (`kubectl`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Honzys/dot-bin](https://github.com/Honzys/dot-bin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
