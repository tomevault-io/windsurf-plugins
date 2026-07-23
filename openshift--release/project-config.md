---
trigger: always_on
description: Validates step registry definitions for correctness.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repository holds OpenShift cluster manifests, component build manifests and CI workflow configuration for OpenShift component repositories for both OKD and OCP.

## Documentation

Additional documentation and references for working with this repository:

- [OpenShift CI Documentation](https://docs.ci.openshift.org/) - Main documentation site for OpenShift CI infrastructure
- [CI Operator Reference](https://steps.ci.openshift.org/ci-operator-reference) - Comprehensive reference for CI operator configuration specification and step registry

## Repository Structure

- `ci-operator/config/` - CI configuration files defining builds and tests for component repositories
- `ci-operator/jobs/` - Generated Prow job configurations (auto-generated, rarely edited manually)
- `ci-operator/step-registry/` - Reusable test steps, chains, and workflows for multi-stage jobs
- `ci-operator/templates/` - Legacy black-box test workflows (deprecated, use step-registry instead)
- `core-services/` - Core service configurations applied to api.ci cluster
- `services/` - Additional service configurations
- `cluster/` - Legacy cluster provisioning manifests
- `projects/` - Experimental, legacy, or non-critical service manifests
- `tools/` - Container image build manifests for tooling

## Slash Commands

This repository includes custom slash commands to improve productivity:

### `/step-finder` - Component Discovery
Search the step-registry (4,400+ reusable CI components) to find existing steps, workflows, and chains before creating new ones:

```bash
/step-finder aws upgrade workflow          # Find AWS upgrade workflows
/step-finder install operator step yes     # Find operator installation steps with usage examples
```

**Always use `/step-finder` before creating new step-registry components to avoid duplication.**

### `/test-repo-files` - Repo File URL Validation
Validate that `core-services/release-controller/_repos/ocp-*.repo` files have correct mirror2 and CDN URLs after editing:

```bash
/test-repo-files 4.22              # Test all 4.22 repo files
/test-repo-files ocp-4.22-rhel9    # Test a specific repo file
/test-repo-files                   # Test all modified repo files vs main
```

See `.claude/SLASH_COMMANDS.md` for detailed documentation.

## Common Development Commands

### Configuration Updates
After modifying CI configuration files, regenerate downstream artifacts:
```bash
make update
```
This runs: `make jobs`, `make ci-operator-config`, `make prow-config`, `make registry-metadata`, `make release-controllers`, `make boskos-config`

### CI Job Configuration
When modifying CI jobs in `ci-operator/config/`:
```bash
make update
```
This validates config, generates Prow job configs, and sanitizes job definitions.

### Validation
```bash
make check              # Validate all service configs
make check-core         # Validate core-services
make check-services     # Validate services
make checkconfig        # Validate Prow configuration
```

### New Repository Setup
```bash
make new-repo
```
Interactive tool to configure CI workflow for a new component repository. Automatically runs `make update` afterwards.

### Step Registry Validation
```bash
make validate-step-registry
```
Validates step registry definitions for correctness.

### Release Controllers
```bash
make release-controllers
```
Regenerates release controller configurations.

## Architecture

### CI Operator Configuration Flow
1. Developers edit YAML files in `ci-operator/config/<org>/<repo>/`
2. Each config file defines:
   - `build_root`: Base image for builds
   - `images`: Container images to build
   - `tests`: Test definitions (can reference step-registry workflows)
   - `promotion`: Where to push successful builds
   - `releases`: OpenShift releases to test against
3. Running `make update` generates Prow job configs in `ci-operator/jobs/`
4. Prow triggers jobs based on PR/merge/periodic events
5. Jobs execute ci-operator with the config, which orchestrates builds and tests

### Step Registry System
The step-registry contains reusable CI components:
- **Steps** (`.yaml` + `-commands.sh`): Atomic tasks (e.g., `openshift-e2e-test`)
- **Chains** (`-chain.yaml`): Ordered sequences of steps
- **Workflows** (`-workflow.yaml`): Complete test scenarios with pre/test/post phases
- **References** (`-ref.yaml`): Step definitions with metadata

Steps are referenced in `ci-operator/config/` test definitions using multi-stage syntax. The registry allows sharing common operations (cluster provisioning, testing, cleanup) across many repositories.

### Configuration Generation Pipeline
1. `ci-operator/config/` is the source of truth (manually edited)
2. `ci-operator-prowgen` generates `ci-operator/jobs/` from config files
3. `determinize-ci-operator` normalizes config file formatting
4. `determinize-prow-config` normalizes Prow configuration
5. All generation uses containerized tools (pulled from quay.io/openshift/ci-public)

### File Naming Conventions
- `ci-operator/config/`: `<org>-<repo>-<branch>.yaml`
- `ci-operator/config/`: `<org>-<repo>-<branch>__periodics.yaml` (variant periodic config)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/release](https://github.com/openshift/release) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
