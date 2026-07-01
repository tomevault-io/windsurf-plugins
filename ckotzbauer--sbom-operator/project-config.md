---
trigger: always_on
description: `sbom-operator` is a Kubernetes operator that catalogues all container images running in a Kubernetes cluster by generating Software Bills of Materials (SBOMs) using [Anchore Syft](https://github.com/anchore/syft). Generated SBOMs are stored to one or more configurable targets: Git repository, Dependency Track, OCI registry, or Kubernetes ConfigMaps.
---

# CLAUDE.md

## Project Overview

`sbom-operator` is a Kubernetes operator that catalogues all container images running in a Kubernetes cluster by generating Software Bills of Materials (SBOMs) using [Anchore Syft](https://github.com/anchore/syft). Generated SBOMs are stored to one or more configurable targets: Git repository, Dependency Track, OCI registry, or Kubernetes ConfigMaps.

The operator watches for pod changes via Kubernetes informers (real-time mode) or runs on a configurable CRON schedule (daemon mode). Already-processed images are tracked via pod annotations (`ckotzbauer.sbom-operator.io/<container-name>`) to avoid redundant scans. Orphan images (no longer running in the cluster) are automatically cleaned up from targets.

- **Module**: `github.com/ckotzbauer/sbom-operator`
- **Container image**: `ghcr.io/ckotzbauer/sbom-operator`
- **License**: (see `/home/christian/Dokumente/dev/github/sbom-git-operator/LICENSE`)

## Tech Stack

| Component                    | Details                                                                            |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| Language                     | Go 1.26.1                                                                          |
| CLI framework                | `github.com/spf13/cobra` v1.10.2                                                   |
| Configuration                | `github.com/ckotzbauer/libstandard` (struct tags: `yaml`, `env`, `flag`)           |
| Kubernetes client            | `k8s.io/client-go` v0.35.0, `k8s.io/api` v0.35.0, `k8s.io/apimachinery` v0.35.0    |
| Kubernetes helpers           | `github.com/ckotzbauer/libk8soci` (pod/image extraction, git operations, OCI auth) |
| SBOM generation              | `github.com/anchore/syft` v1.42.1, `github.com/anchore/stereoscope` v0.1.20        |
| Dependency Track client      | `github.com/DependencyTrack/client-go` v0.18.0                                     |
| OCI registry                 | `github.com/google/go-containerregistry` v0.21.2                                   |
| Docker image parsing         | `github.com/novln/docker-parser` v1.0.0                                            |
| CRON scheduling              | `github.com/robfig/cron` v1.2.0                                                    |
| Logging                      | `github.com/sirupsen/logrus` v1.9.4                                                |
| Testing                      | `github.com/stretchr/testify` v1.11.1                                              |
| SQLite (Syft RPM cataloging) | `modernc.org/sqlite` v1.44.3                                                       |
| GCP auth                     | `golang.org/x/oauth2/google`                                                       |
| Build                        | GoReleaser v2 (`.goreleaser.yml`)                                                  |
| Linting                      | golangci-lint v2.5.0, gosec v2.22.9                                                |
| Signing                      | cosign (release artifacts and container images)                                    |
| Dependency management        | Renovate (extends `ckotzbauer/renovate-config`)                                    |

## Project Structure

```
sbom-git-operator/
├── main.go                          # Entrypoint, CLI flag definitions, health endpoint (:8080)
├── internal/
│   ├── config.go                    # Config struct with yaml/env/flag tags, config key constants
│   ├── daemon/
│   │   └── daemon.go                # CRON-based background service (mutex-protected re-entrance guard)
│   ├── job/
│   │   └── job.go                   # Kubernetes Job creation for delegated SBOM generation
│   ├── kubernetes/
│   │   ├── kubernetes.go            # KubeClient: pod/namespace informers, annotations, jobs, configmaps
│   │   ├── image.go                 # Registry proxy logic (ApplyProxyRegistry)
│   │   └── image_test.go            # Tests for proxy registry mapping
│   ├── processor/
│   │   └── processor.go             # Core orchestration: pod watching, SBOM scanning, target dispatch
│   ├── syft/
│   │   ├── syft.go                  # Syft integration: SBOM creation, format encoding, GCP Workload Identity
│   │   └── syft_test.go             # Integration tests against real container images (alpine, redis, node, fedora)
│   └── target/
│       ├── target.go                # Target interface definition (Initialize, ValidateConfig, ProcessSbom, LoadImages, Remove)
│       ├── git/
│       │   ├── git_target.go        # Git target: stores SBOMs as files in a git repository
│       │   └── git_target_test.go   # Tests for ImageIDToFilePath
│       ├── dtrack/
│       │   └── dtrack_target.go     # Dependency Track target: uploads BOMs via API, manages project tags
│       ├── oci/
│       │   ├── oci_target.go        # OCI registry target: pushes SBOMs as OCI artifacts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ckotzbauer/sbom-operator](https://github.com/ckotzbauer/sbom-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
