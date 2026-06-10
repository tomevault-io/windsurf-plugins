---
trigger: always_on
description: - Canonical policy and constraints: this file
---

# GitHub Copilot Instructions for Pipeleek

## Maintainer Quick Index

- Canonical policy and constraints: this file
- Maintainer quick-start router: `.instructions.md`
- Agent mode definitions: `.github/AGENTS.md`
- Reusable task skills:
    - `.github/SKILLS/command-implementation.md`
    - `.github/SKILLS/command-coverage.md`
    - `.github/SKILLS/docs-sync.md`
    - `.github/SKILLS/new-command.md`
    - `.github/SKILLS/pr-review-fixes.md`
    - `.github/SKILLS/pr-actions-failures-debug.md`

## Project Overview

Pipeleek is a CLI tool designed to scan CI/CD logs and artifacts for secrets across multiple platforms including GitLab, GitHub, BitBucket, Azure DevOps, and Gitea. The tool uses TruffleHog for secret detection and provides additional helper commands for exploitation workflows.

## Technology Stack

- **Language**: Go 1.24+
- **CLI Framework**: Cobra (github.com/spf13/cobra)
- **Logging**: Zerolog (github.com/rs/zerolog)
- **Secret Detection**: TruffleHog v3
- **Testing**: Go testing framework with testify
- **Build Tool**: Go build system
- **Release**: GoReleaser

## Project Structure

```
pipeleek/
├── cmd/pipeleek/           # CLI entry point (main.go)
├── internal/cmd/           # CLI commands (using Cobra) - internal package
│   ├── bitbucket/          # BitBucket-specific commands
│   ├── devops/             # Azure DevOps commands
│   ├── docs/               # Documentation command
│   ├── flags/              # Common CLI flags
│   ├── gitea/              # Gitea commands
│   ├── github/             # GitHub-specific commands
│   ├── gitlab/             # GitLab-specific commands
│   ├── root.go             # Root command definition
│   └── root_test.go        # Root command tests
├── pkg/                    # Core business logic packages
│   ├── archive/            # Archive handling
│   ├── bitbucket/          # BitBucket business logic
│   ├── config/             # Configuration management
│   ├── devops/             # Azure DevOps business logic
│   ├── docs/               # Documentation generation
│   ├── format/             # Formatting helpers
│   ├── gitea/              # Gitea business logic
│   ├── github/             # GitHub business logic
│   ├── gitlab/             # GitLab business logic
│   ├── httpclient/         # HTTP client helpers
│   ├── logging/            # Logging helpers
│   ├── scan/               # Scan logic
│   ├── scanner/            # Scanner engine
│   └── system/             # System helpers
├── tests/e2e/              # End-to-end tests
├── docs/                   # Documentation (MkDocs)
├── .github/                # GitHub workflows and configs
│   └── workflows/          # CI/CD pipelines
├── go.mod                  # Go module definition (at root)
├── go.sum                  # Dependency checksums
├── Makefile                # Build and test commands
└── goreleaser.yaml         # Release configuration
```

## Building and Testing

### Building the Project

```bash
make build
# Or directly:
go build -o pipeleek ./cmd/pipeleek
```

### Running Tests

**Using Makefile (recommended):**

```bash
make test           # Run all tests (unit + e2e)
make test-unit      # Run unit tests only
make test-e2e       # Run all e2e tests
```

**Unit tests (excluding e2e):**

```bash
make test-unit
# Or directly:
go test $(go list ./... | grep -v /tests/e2e) -v -race
```

**End-to-end tests:**

E2E tests are organized by platform in a structured folder hierarchy:

```
tests/e2e/
├── gitlab/          # GitLab-specific tests
│   ├── cicd/yaml/   # CICD YAML command tests
│   ├── scan/        # Scan command tests
│   ├── variables/   # Variables command tests
│   ├── schedule/    # Schedule command tests
│   ├── runners/     # Runners list/exploit tests
│   ├── secureFiles/ # Secure files tests
│   ├── vuln/        # Vulnerability check tests
│   ├── renovate/    # Renovate tests
│   └── unauth/      # Unauthenticated commands (shodan)
├── github/          # GitHub Actions tests
├── bitbucket/       # BitBucket tests
├── devops/          # Azure DevOps tests
├── gitea/           # Gitea tests
└── internal/        # Shared test utilities
    └── testutil/    # Common helpers (RunCLI, mock servers, etc.)
```

**Using Makefile (recommended):**

```bash
make test-e2e              # Run all e2e tests
make test-e2e-gitlab       # Run only GitLab e2e tests
make test-e2e-github       # Run only GitHub e2e tests
make test-e2e-bitbucket    # Run only BitBucket e2e tests
make test-e2e-devops       # Run only Azure DevOps e2e tests
make test-e2e-gitea        # Run only Gitea e2e tests
```

**Manual execution:**
To run e2e tests manually, first build the binary and set `PIPELEEK_BINARY`:

```bash
go build -o pipeleek ./cmd/pipeleek
PIPELEEK_BINARY=$(pwd)/pipeleek go test ./tests/e2e/... -tags=e2e -v -timeout 10m
```

Run tests for a specific platform:

```bash
# GitLab tests only
PIPELEEK_BINARY=$(pwd)/pipeleek go test ./tests/e2e/gitlab/... -tags=e2e -v

# Specific command tests
PIPELEEK_BINARY=$(pwd)/pipeleek go test ./tests/e2e/gitlab/scan -tags=e2e -v
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CompassSecurity/pipeleek](https://github.com/CompassSecurity/pipeleek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
