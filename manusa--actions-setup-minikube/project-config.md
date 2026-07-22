---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# Setup Minikube GitHub Action - AI Agents Instructions

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

This file provides guidance to AI coding agents (GitHub Copilot, Claude Code, etc.) when working with code in this repository.

## Project Overview

A GitHub Action that sets up a single-node Kubernetes cluster using Minikube in CI workflows. It downloads and installs Minikube with specified versions of Kubernetes, supporting multiple drivers (`none`, `docker`) and container runtimes (`docker`, `cri-o`, `containerd`). Built with Node.js 24 using GitHub Actions toolkit libraries.

## Working Effectively

### Bootstrap and Setup
```shell
npm install
```

### Build Commands

This is a GitHub Action - no build step required. The action runs directly from `src/index.js`.

### Testing

**Unit tests (fast, ~2s):**
```shell
npm test
```

**Format check:**
```shell
npm run format-check
```

**IMPORTANT**: E2E tests run only in GitHub Actions workflows. They cannot be run locally as they require the GitHub Actions environment and actually provision Minikube clusters.

### Running the Application

This is a GitHub Action and cannot be run directly. Test locally by:
1. Running unit tests: `npm test`
2. Pushing to a branch and observing the CI workflow

## Architecture

### Technical Structure

```
src/
  index.js              # Entry point - orchestrates the setup process
  check-environment.js  # Validates Ubuntu version (18, 20, 22, 24)
  check-kubernetes-version.js # Validates K8s version against Minikube's supported list
  configure-environment.js # Prepares system (apt packages, Docker, CNI plugins)
  download.js           # Downloads binaries from GitHub releases (Minikube, CNI plugins, crictl, cri-dockerd)
  error-handler.js      # Global error handling
  exec.js               # Shell command execution utilities
  github.js             # GitHub API request utility (authenticated/unauthenticated)
  install.js            # Installs and starts Minikube
  load-inputs.js        # Loads action inputs via @actions/core
  __tests__/            # Jest unit tests (mirror src/ structure)

action.yml              # GitHub Action definition (outputs: `force`)
.github/workflows/
  check.yml             # CI: format check + unit tests
  runner.yml            # E2E tests: runs action against multiple K8s versions
```

### Design Patterns

- **Modular pipeline**: `index.js` orchestrates: `checkEnvironment()` → `loadInputs()` → `configureEnvironment(inputs)` → `download.downloadMinikube(inputs)` → `install(downloadedFile, inputs)`. Note: binary downloads for CNI plugins, crictl, and cri-dockerd happen inside `configureEnvironment()`, not as a separate pipeline step.
- **GitHub Actions toolkit**: Uses `@actions/core` for inputs/outputs, `@actions/tool-cache` for downloads
- **GitHub API integration**: `src/github.js` provides a `gitHubRequest` utility wrapping Axios for authenticated/unauthenticated GitHub API calls. Used by `download.js` and `check-kubernetes-version.js`.
- **Driver-specific logic**: Different setup paths for `none` vs `docker` drivers (none requires CNI plugins, crictl, cri-dockerd)
- **Kubernetes version validation**: `check-kubernetes-version.js` checks if the requested K8s version is in Minikube's built-in supported list. If not, it verifies the version exists as a GitHub release and returns `UNSUPPORTED` (triggering `--force` flag). If the version doesn't exist at all, it throws an error.

### Key Dependencies

**npm packages (in `package.json`):**
- `@actions/core` - Action inputs, outputs, and logging
- `@actions/github` - GitHub context utilities
- `@actions/io` - File system operations
- `@actions/tool-cache` - Binary downloads and caching
- `axios` - HTTP requests to GitHub API (via `src/github.js`)

**Binary dependencies (pinned versions in `src/download.js`):**
- **CNI plugins** (`containernetworking/plugins`) - Required by cri-dockerd and recent Minikube releases for container networking
- **cri-tools / crictl** (`kubernetes-sigs/cri-tools`) - CRI CLI tool for interacting with container runtimes
- **cri-dockerd** (`Mirantis/cri-dockerd`) - CRI shim for Docker Engine

These binaries are downloaded at runtime from GitHub releases. Their versions are hardcoded as `const tag = '...'` values in `src/download.js` (not in `package.json`).

### SHA256 Verification

Every binary downloaded by `src/download.js` is SHA256-verified before use. Two helpers funnel all downloads — **no bare `tc.downloadTool` call should appear in this module**:

- **`downloadGitHubArtifact(...)`** — for release-asset downloads. Requires exactly one of:
  - **`verifyWithCompanionSha256: true`** — looks up the `<asset.name>.sha256` companion asset in the same release, fetches its body via `gitHubRequest` (with `responseType: 'text'`), parses the leading hex token, validates the format with `assertSha256Hex`, and aborts on mismatch. Used for **minikube**, **CNI plugins**, and **crictl** — all three upstreams publish `.sha256` companions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [manusa/actions-setup-minikube](https://github.com/manusa/actions-setup-minikube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
