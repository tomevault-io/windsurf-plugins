---
trigger: always_on
description: **ALWAYS follow these instructions and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**
---

# GitHub Copilot Instructions for Kubernoodles

**ALWAYS follow these instructions and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.**

This file provides instructions to GitHub Copilot for working effectively with the Kubernoodles repository.

## Repository Overview

Kubernoodles is a framework for managing custom self-hosted runners for GitHub Actions in Kubernetes at enterprise scale. This repository contains:

- **Docker images**: Located in `/images/` - Contains Dockerfiles for various runner images
- **Kubernetes deployments**: Located in `/deployments/` - Kubernetes manifests for deploying runners
- **Cluster configs**: Located in `/cluster-configs/` - Configuration files for different cluster setups
- **Documentation**: Located in `/docs/` - Comprehensive guides and troubleshooting
- **Tests**: Located in `/tests/` - End-to-end tests for validating runner images

## Working Effectively with the Codebase

### Development Environment Setup

Do NOT try to set up a traditional local development environment. This is an infrastructure project that requires:
- Docker for building container images
- Access to external container registries (docker.io, quay.io, ghcr.io, registry.access.redhat.com, cgr.dev)
- Kubernetes cluster for testing deployments
- GitHub App credentials for runner registration

### Building and Testing

#### **CRITICAL BUILD TIMING**
- **NEVER CANCEL builds or long-running commands** - Docker image builds take 15-45 minutes per image
- Set timeouts to 60+ minutes for build commands
- Multi-architecture builds (amd64 + arm64) take significantly longer
- Test workflows take 15+ minutes (build→deploy→test→cleanup cycle)

#### Build Commands
```bash
# Build a specific image (from repository root)
docker build -f images/ubi8.Dockerfile -t test-runner:ubi8 .

# Build with multi-architecture support
docker buildx build --platform linux/amd64,linux/arm64 -f images/ubi8.Dockerfile -t test-runner:ubi8 .

# Build all images (use the GitHub workflow instead)
# See .github/workflows/build-latest.yml
```

#### Available Images

Build any of these Dockerfiles in `/images/` (6 main production images):
- `ubi8.Dockerfile` - Red Hat UBI 8 (rootful, no sudo)
- `ubi9.Dockerfile` - Red Hat UBI 9 (rootful, no sudo)  
- `ubi10.Dockerfile` - Red Hat UBI 10 (rootful, no sudo)
- `rootless-ubuntu-jammy.Dockerfile` - Ubuntu 22.04 LTS with rootless Docker-in-Docker
- `rootless-ubuntu-numbat.Dockerfile` - Ubuntu 24.04 LTS with rootless Docker-in-Docker  
- `wolfi.Dockerfile` - Chainguard Wolfi (minimal, no sudo)

**Additional specialized images:**
- `ghes-demo.Dockerfile` - Demo image for GitHub Enterprise Server.  Do not update or change anything in this file.
- `kaniko-build-test.Dockerfile` - Test image for Kaniko builds

### Linting and Validation

**ALWAYS validate changes before committing** using these commands:

```bash
# 1. Shell script linting (if modifying .sh files)
shellcheck images/*.sh
shellcheck images/software/*.sh

# 2. The repository uses Super Linter via GitHub Actions
# Trigger manually via GitHub UI or by creating PR
# See .github/workflows/super-linter.yml

# 3. Validate JSON files (if modifying deployments)
find . -name "*.json" -exec python -m json.tool {} \;

# 4. Basic Dockerfile syntax validation (will fail fast if major syntax errors)
docker build -f images/ubi8.Dockerfile -t syntax-test . --target build 2>&1 | head -10
```

**Linting Configuration:**
- `.github/linters/.hadolint.yaml` - Dockerfile linting rules
- `.github/linters/.markdownlint.json` - Markdown formatting rules

**The super-linter workflow validates:**
- Dockerfile syntax and best practices (Hadolint)
- Markdown formatting and links  
- JSON syntax validation
- Additional code quality checks

### Testing Strategy

This project uses **End-to-End testing only** - no unit tests. The test cycle is:
1. Build Docker image and push to registry (15-45 minutes)
2. Deploy to Kubernetes cluster as GitHub Actions runner (5-10 minutes)
3. Run tests as GitHub Actions workflows (5-10 minutes)  
4. Clean up deployment (2-5 minutes)

**Total test cycle time: 25-70 minutes per image. NEVER CANCEL.**

#### Test Workflow Structure
```bash
# Each test workflow follows this pattern:
.github/workflows/test-{image}.yml
├── build job    # Build and push test image
├── deploy job   # Deploy to test Kubernetes namespace  
├── test job     # Run actual tests (timeout: 15 minutes)
└── cleanup job  # Remove deployment (runs even if tests fail)
```

#### Manual Validation Scenarios

**After modifying Docker images:**
1. Build the specific image locally (expect 15-45 minutes)
2. Verify the image starts without errors
3. Test that required software is installed correctly
4. For rootless images: Verify Docker daemon starts in rootless mode
5. For UBI images: Verify Podman/Buildah/Skopeo work correctly

**After modifying deployments:**
1. Deploy to test Kubernetes cluster
2. Verify runner registers with GitHub successfully
3. Run a basic workflow to test functionality
4. Check resource limits and requests are appropriate

**After modifying tests:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [some-natalie/kubernoodles](https://github.com/some-natalie/kubernoodles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
