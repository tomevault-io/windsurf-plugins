---
trigger: always_on
description: This file contains project-specific guidelines for GitHub Copilot to ensure code suggestions align with Blue Agent's mandatory conventions and best practices.
---

# GitHub Copilot Instructions for Blue Agent

This file contains project-specific guidelines for GitHub Copilot to ensure code suggestions align with Blue Agent's mandatory conventions and best practices.

## Project Overview

Blue Agent is a self-hosted Azure Pipelines agent designed to run in Kubernetes environments. The project aims to provide a cost-effective, secure, auto-scaled, and easy-to-deploy alternative to Microsoft-hosted agents.

### Key Features and Motivations

- **Cost Efficiency**: Dynamic provisioning with KEDA auto-scaling (0 to 100+ agents in seconds)
- **Performance**: Customizable performance beyond Microsoft-hosted agent limitations
- **Security**: SBOM generation, Cosign signing, daily system updates, air-gapped capability
- **Flexibility**: Multi-OS support (Azure Linux, Debian, Ubuntu, RHEL, Windows Server)
- **Container Building**: Built-in BuildKit support for container builds
- **Self-Management**: Agent auto-registration and restart capabilities

### Core Architecture Components

- **Agent Runtime**: Multi-flavor container images with pre-installed tools (jq, PowerShell, Python 3, etc.)
- **Template Job System**: Special "template" containers (`AZP_TEMPLATE_JOB=1`) that register briefly to establish KEDA scaling capabilities
- **Multi-Deployment Target**: Both Kubernetes (via Helm) and Azure Container Apps (via Bicep)
- **Build Orchestration**: Make-based workflow with environment variable injection from `cicd/env-github-actions.sh`
- **Integration Testing**: Parallel test execution against live Azure DevOps pools using GNU parallel

### General Constraints

- **Container-First**: All deployments target container environments including Kubernetes and Azure Container Apps
- **Security-First**: All images must be signed, include SBOM, and follow SLSA 4 requirements
- **Multi-Architecture**: Support for both amd64 and arm64 architectures is mandatory
- **Cloud-Native**: Designed for cloud-native environments with KEDA integration
- **Documentation-Driven**: All features must be documented and examples provided

## Developer Workflows

### Build System (Make-based)

The project uses Make as the primary build orchestrator. Key commands:

```bash
# Local multi-flavor Docker builds
make build-docker flavor=bookworm version=latest

# Full deployment with Bicep (Azure Container Apps)
make deploy-bicep flavor=bookworm version=1.0.0

# Integration testing
make integration prefix=test flavor=bookworm version=1.0.0
```

Environment variables are centralized in `cicd/env-github-actions.sh` - always source this file when extending build scripts.

### Multi-Flavor Container Strategy

All Dockerfiles follow the pattern `src/docker/Dockerfile-{flavor}` where flavor corresponds to OS distributions:

- `azurelinux3`, `bookworm`, `jammy`, `noble` (Linux variants)
- `ubi8`, `ubi9` (Red Hat Enterprise Linux)
- `win-ltsc2022`, `win-ltsc2025` (Windows Server)

Build argument pattern is consistent across all flavors - check `cicd/docker-build-local.sh` for the canonical build argument list.

### Template Job Pattern

When developing KEDA integration features, understand the template job concept:

- Template containers run with `AZP_TEMPLATE_JOB=1`
- They register with Azure DevOps, establish capabilities, then exit after 60 seconds
- KEDA uses template agents as scaling references when no active agents exist
- See `src/docker/start.sh` lines 1-50 for the template job logic

### Integration Testing Architecture

Tests run in parallel using GNU parallel against live Azure DevOps organization:

- Organization: `https://dev.azure.com/blue-agent`
- Test pipeline definitions in `test/pipeline/*.yaml`
- Agent pool validation in `test/azure-devops/template-exists.sh`
- Cleanup verification in `test/azure-devops/queue-cleaned.sh`

## Security and Compliance

### SLSA 4 Supply Chain Requirements

- All builds must generate SLSA provenance attestations
- Use `--provenance=true` flag in Docker build commands
- Include SBOM generation with `--sbom=true`
- Example: `docker build --provenance=true --sbom=true --tag $IMAGE .`
- Store provenance in container registry alongside images

### Secrets Management

- Never hardcode secrets in configuration files
- Use SOPS for encrypted configuration: `sops -e config.yaml > config.enc.yaml`
- Azure Key Vault integration available in Helm chart (`secret.azureKeyVault.enabled`)
- Prefer environment variables over files for secret injection

### Multi-arch Support

- All container builds MUST support both amd64 and arm64 architectures
- Use `--platform=linux/amd64,linux/arm64` in Docker buildx commands
- Test both architectures in CI/CD pipeline
- Example: `docker buildx build --platform=linux/amd64,linux/arm64 --tag $IMAGE .`

## CI/CD and Azure Pipelines

### Azure Pipelines Target

- All CI snippets and examples MUST target Azure Pipelines
- Use YAML pipeline syntax, not classic editor
- Include KEDA auto-scaling configuration in pipeline examples
- Example pool configuration:

```yaml
pool:
  name: "my-agent-pool"
  demands:
    - agent.name -equals blue-agent
```

### KEDA Auto-scaling

- Include KEDA ScaledJob configuration in Kubernetes examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clemlesne/blue-agent](https://github.com/clemlesne/blue-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
