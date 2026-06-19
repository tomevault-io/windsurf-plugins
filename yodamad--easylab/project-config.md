---
trigger: always_on
description: EasyLab is a Go web application that provisions cloud-based lab environments using Pulumi IaC on OVHcloud. It deploys Kubernetes clusters with Coder workspaces for workshop participants.
---

# EasyLab — Claude Code Rules

## Project Overview

EasyLab is a Go web application that provisions cloud-based lab environments using Pulumi IaC on OVHcloud. It deploys Kubernetes clusters with Coder workspaces for workshop participants.

Two main entry points:
1. A Pulumi program (`main.go`) for infrastructure provisioning
2. A web server (`cmd/server/main.go`) with an HTMX-powered admin/student UI

### Tech Stack

- Go 1.26 with `html/template` for server-side rendering
- HTMX for dynamic UI interactions (no frontend framework)
- Pulumi SDK v3 for infrastructure as code (Go-based programs)
- OVHcloud provider for cloud resources (network, k8s, node pools)
- Coder SDK for workspace and template management
- `net/http` with `http.ServeMux` for routing (no external router)
- Playwright for E2E frontend testing
- Makefile for build, test, and CI tasks
- Docker + docker-compose for containerized deployment
- Kustomize manifests for Kubernetes deployment (`k8s-deployment/`)
- MkDocs for documentation (`docs/`)

### Folder Structure

```
main.go                  # Pulumi IaC entry point (OVH infra + k8s + Coder setup)
cmd/
  server/
    main.go              # Web server entry point (HTTP server with graceful shutdown)
internal/
  server/
    handler.go           # HTTP handlers (admin UI, student portal, credentials, labs)
    auth.go              # Authentication (admin password, student login, JWT cookies)
    credentials.go       # Provider credentials management (OVH, extensible)
    job.go               # Job/lab lifecycle management (create, status, persist)
    pulumi.go            # Pulumi executor (preview, execute, destroy, retry)
    cleanup.go           # Background workspace cleanup goroutine (every 5 min)
    feedback.go          # Student feedback form + FeedbackStore (JSON persistence)
    github.go            # GitHub API — fetch latest stable Coder releases
    azure_api.go         # Azure-specific admin API handlers
    azure_options.go     # Azure AD runtime config + AzureADConfig struct
    ovh_api.go           # OVH-specific admin API handlers
    ovh_options.go       # OVH options cache (regions, VM sizes) with admin filtering
  providers/
    provider.go          # Cloud provider interface
    registry.go          # Provider registry
    ovh/                 # OVH-specific provider implementation
    azure/               # Azure-specific provider implementation
    dns/
      provider.go        # DNS provider interface (GetCredentialFields, SetupCertManagerDNS01, CreateARecord)
      registry.go        # DNS provider registry
      ovh/               # OVH DNS implementation
      azure/             # Azure DNS implementation
  tfparse/
    variables.go         # Terraform .tf file parser — extracts variable definitions from ZIP uploads
  pulumi/
    program.go           # Pulumi program builder
coder/
  coder.go               # Coder API client (users, workspaces, templates)
  https.go               # HTTPS/TLS setup: ingress-nginx, cert-manager, DNS-01/HTTP-01 ACME
k8s/
  k8s.go                 # Kubernetes provider, namespace, external IP
  helm.go                # Helm release management
ovh/
  ovh.go                 # OVH resource provisioning (network, k8s cluster, node pools)
utils/
  config.go              # Pulumi config helpers and constants
  git.go                 # Git clone, zip utilities
  log.go                 # Logging helpers
templates/               # Pulumi template code (copied into job workspaces)
web/
  base.html              # Shared HTML base template
  static/
    style.css            # Global styles
    *.js                 # Page-specific JS (HTMX interactions)
tests/
  *.spec.ts              # Playwright E2E tests
  chaos/                 # Chaos testing specs
docs/                    # MkDocs documentation source
Makefile                 # Build, test, coverage, CI targets
```

---

## Guardrails

CRITICAL: Do NOT modify, refactor, rename, or restructure existing files unless the user EXPLICITLY asks for it. This applies to all code, tests, configs, and documentation. When in doubt, ASK before changing.

### Must NOT

- Modify files not directly related to the user's request
- Refactor or "improve" existing working code on own initiative
- Rename functions, variables, types, packages, or files without being asked
- Change API routes, request/response contracts, or HTTP status codes
- Alter Pulumi infrastructure code (`main.go`, `ovh/`, `k8s/`, `coder/`, `templates/`)
- Edit CI/CD pipelines (`.gitlab-ci.yml`, `.github/`), `Dockerfile`, or `docker-compose.yml`
- Add, remove, or upgrade Go or npm dependencies without explicit approval
- Change authentication or security logic (`internal/server/auth.go`)
- Modify the Makefile targets or build configuration
- Move files between directories or reorganize the project structure
- Remove backward-compatibility routes or aliases

### Should

- Limit changes to the minimum scope needed to fulfill the request
- Explain what is intended to change BEFORE making edits to multiple files
- Preserve existing code style, naming conventions, and patterns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yodamad/easylab](https://github.com/yodamad/easylab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
