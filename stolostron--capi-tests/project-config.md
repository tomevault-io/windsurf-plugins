---
trigger: always_on
description: This file provides guidance to Gemini when working with code in this repository.
---

# GEMINI.md

This file provides guidance to Gemini when working with code in this repository.

## Repository Purpose

This is a Go-based CAPI test suite, currently supporting CAPZ/ARO and CAPA/ROSA paths. The tests verify the complete deployment workflow from prerequisites to cluster verification.

## Core Principles

- **Source of Truth**: This `GEMINI.md` file is the single source of truth for all project-specific guidelines. The instructions herein must be strictly followed, overriding any conflicting general instructions or prior knowledge.
- **Code Reviews**: When asked to perform a code review, NEVER implement changes directly. First, suggest any changes and ask for explicit permission from the user before proceeding with implementation.

## Test Architecture

### Test Execution Model

Tests are designed to run **sequentially** in a specific order, with each phase depending on the previous phase's success:

1. **Check Dependencies** (`01_check_dependencies_test.go`) - Tool availability and authentication
2. **Setup** (`02_setup_test.go`) - Repository cloning and validation
3. **Kind Cluster** (`03_cluster_test.go`) - Management cluster deployment
4. **Infrastructure** (`04_generate_yamls_test.go`) - YAML generation
5. **Deployment** (`05_deploy_crs_test.go`) - CR deployment monitoring
6. **Verification** (`06_verification_test.go`) - Final cluster validation

Tests are **idempotent** - they skip steps already completed, allowing re-runs.

### Configuration System

All test configuration is centralized in `test/config.go` via the `TestConfig` struct. Configuration follows this precedence:

1. Environment variables (highest priority)
2. Defaults in `NewTestConfig()`

Key configuration pattern:
```go
config := NewTestConfig()  // Creates config with env vars or defaults
```

Never hardcode values - always use `GetEnvOrDefault()` for new configuration.

### Helper Functions

`test/helpers.go` provides shared utilities used across all tests:

- `CommandExists(cmd)` - Check if CLI tool is available
- `RunCommand(t, name, args...)` - Execute shell commands with test context
- `SetEnvVar(t, key, value)` - Set env var with automatic cleanup
- `FileExists(path)` / `DirExists(path)` - Path validation
- `GetEnvOrDefault(key, default)` - Config value resolution

Always use these helpers instead of reimplementing functionality.

### Test Patterns

All test functions follow this pattern:
```go
func TestPhase_Specific(t *testing.T) {
    config := NewTestConfig()

    // Validate prerequisites
    if !prerequisitesMet {
        t.Skipf("Prerequisites not met")
    }

    // Perform test action
    // Use t.Logf() for progress
    // Use t.Errorf() for non-fatal errors
    // Use t.Fatalf() for fatal errors that prevent continuation
}
```

## Development Commands

### Running Tests

```bash
# Check dependencies tests only (fast, no Azure resources)
make test

# Full test suite (all phases sequentially)
make test-all

# Individual test phases (internal use - called by test-all)
make _check-dep      # Check dependencies
make _setup          # Repository setup
make _management_cluster # Cluster deployment
make _generate-yamls # YAML generation
make _deploy-crs     # CR deployment
make _verify-workload-cluster  # Workload cluster verification
make _delete-workload-cluster  # Workload cluster deletion
make _validate-cleanup         # Cleanup validation

# Run specific test function
go test -v ./test -run TestCheckDependencies_ToolAvailable
go test -v ./test -run TestInfrastructure

# With custom configuration
DEPLOYMENT_ENV=prod WORKLOAD_CLUSTER_NAME=my-cluster go test -v ./test -timeout 60m
```

### Repository Management

```bash
# Check prerequisites
make check-prereq

# Setup cluster-api-installer as submodule
make setup-submodule

# Update submodule
make update-submodule

# Clean up test resources (interactive - prompts before deleting each resource)
make clean
```

The `make clean` command is interactive and will prompt you to confirm deletion of:
- Kind cluster
- Cluster-api-installer repository clone
- Kubeconfig files
- Results directory

This prevents accidental deletion and allows selective cleanup.

### Code Quality

```bash
# Format code
make fmt

# Run linters
make lint

# Download/update dependencies
make deps
```

## Integration with cluster-api-installer

Tests require access to the [cluster-api-installer](https://github.com/RadekCap/cluster-api-installer) repository. Three approaches are supported:

1. **Git Submodule** (recommended for development)
   ```bash
   make setup-submodule
   export ARO_REPO_DIR="$(pwd)/vendor/cluster-api-installer"
   ```

2. **Automatic Clone** (CI/CD, default)
   - Tests auto-clone to `/tmp/cluster-api-installer-aro`
   - No manual setup needed

3. **Existing Clone** (manual)
   ```bash
   export ARO_REPO_DIR="/path/to/cluster-api-installer"
   ```

See `docs/INTEGRATION.md` for detailed integration patterns.

## Environment Variables

### Repository Configuration
- `ARO_REPO_URL` - cluster-api-installer URL (default: RadekCap/cluster-api-installer)
- `ARO_REPO_BRANCH` - Branch to use (default: `ARO-ASO`)
- `ARO_REPO_DIR` - Local path (default: `/tmp/cluster-api-installer-aro`)

### Infrastructure Provider

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stolostron/capi-tests](https://github.com/stolostron/capi-tests) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
