---
trigger: always_on
description: GitOps Promoter is a Kubernetes operator that facilitates environment promotion for config managed via GitOps. It provides a drift-free promotion process with a robust gating system, complete integration with git and SCM tooling.
---

# GitOps Promoter - GitHub Copilot Instructions

## Project Overview

GitOps Promoter is a Kubernetes operator that facilitates environment promotion for config managed via GitOps. It provides a drift-free promotion process with a robust gating system, complete integration with git and SCM tooling.

### Key Technologies
- **Backend**: Go 1.25+ (Kubernetes operator using controller-runtime)
- **Frontend**: TypeScript/React (Dashboard UI and Argo CD extension)
- **Infrastructure**: Kubernetes, Argo CD integration
- **SCM Support**: GitHub, GitHub Enterprise, GitLab, Forgejo (including Codeberg)

## Architecture

The project follows a standard Kubernetes operator pattern with:
- Custom Resource Definitions (CRDs) in `api/v1alpha1/`
- Controllers in `internal/controller/`
- SCM integrations in `internal/scms/`
- Webhook receiver for SCM events in `internal/webhookreceiver/`
- Dashboard UI in `ui/dashboard/`
- Argo CD extension in `ui/extension/`
- Shared UI components in `ui/components-lib/`

## Development Setup

### Prerequisites
- Go 1.24+
- Node.js (for UI development)
- kubectl
- Docker or compatible container tool
- Kind (for local testing)

### Build Commands
```bash
# Go backend
make build              # Build manager binary
make build-all         # Build UI components and manager binary
make test              # Run tests
make test-parallel     # Run tests in parallel
make lint              # Run linters
make lint-fix          # Run linters with auto-fix

# UI development
make build-dashboard   # Build dashboard UI
make build-extension   # Build Argo CD extension
make lint-dashboard    # Lint dashboard
make lint-extension    # Lint extension
make lint-ui           # Lint all UI components
make generate-ui-types # Regenerate view APIService TypeScript types (after make generate-apiserver)

# Running locally
make run               # Run controller locally
make run-dashboard     # Run dashboard locally
```

## Code Style and Standards

### Go Code
- Use `go fmt` for formatting
- Follow golangci-lint rules (see `.golangci.yml`)
- Use Ginkgo/Gomega for testing
- Tests should use table-driven tests where appropriate
- Use controller-runtime patterns for Kubernetes interactions
- Avoid naked returns and follow error wrapping conventions

### TypeScript/React Code
- Use TypeScript for all new code
- Follow ESLint rules
- Run type-checking with `tsc --noEmit`
- Use functional components with hooks



## Directory Structure

### Key Directories
- `api/v1alpha1/` - Kubernetes CRD types (PromotionStrategy, GitRepository, ScmProvider, etc.)
- `internal/controller/` - Kubernetes controllers
- `internal/scms/` - SCM provider implementations (GitHub, GitLab, Forgejo)
- `internal/git/` - Git operations
- `internal/utils/` - Shared utilities
- `internal/webhookreceiver/` - Webhook handling for SCM events
- `internal/webserver/` - Dashboard web server
- `config/` - Kubernetes manifests and Kustomize configurations
- `docs/` - Documentation (MkDocs format)
- `hack/` - Development scripts
- `test/` - Test fixtures and e2e tests
- `ui/` - Frontend code

### Generated Files
These files are auto-generated and should not be edited manually:
- `api/v1alpha1/zz_generated.deepcopy.go`
- `config/crd/bases/*.yaml`
- Files in `dist/`

## Testing Guidelines

### Unit Tests
- Use Ginkgo/Gomega for Go tests
- Test files should be named `*_test.go`
- Use `Context` and `It` blocks for test organization
- Mock external dependencies
- Use `envtest` for controller testing

### Running Tests
```bash
make test              # Run all tests
make test-parallel     # Run tests in parallel (faster)
make test-e2e          # Run end-to-end tests
```

### Test Patterns
- Use table-driven tests for multiple similar test cases
- Test both success and error paths
- Use `Eventually` for async operations in controller tests
- Clean up resources in `AfterEach` blocks

## Custom Resources

### Core CRDs
1. **PromotionStrategy** - Defines promotion flow between environments
2. **GitRepository** - Represents a Git repository
3. **ScmProvider** - SCM provider configuration (GitHub, GitLab, Forgejo)
4. **ClusterScmProvider** - Cluster-scoped SCM provider configuration
5. **PullRequest** - Represents a promotion pull request
6. **CommitStatus** - Commit status tracking
7. **ArgoCDCommitStatus** - Argo CD-specific commit status
8. **ChangeTransferPolicy** - Controls how changes are transferred between branches
9. **RevertCommit** - Represents a revert operation
10. **ControllerConfiguration** - Configuration for the GitOps Promoter controller

### Resource Relationships
- PromotionStrategy references GitRepository
- GitRepository references ScmProvider or ClusterScmProvider
- ScmProvider references a Secret for credentials
- ArgoCDCommitStatus references PromotionStrategy

## Common Patterns

### Controller Patterns
- Use structured logging with `logr`
- Return `ctrl.Result{RequeueAfter: duration}` for retries
- Use conditions to track resource status
- Handle resource not found errors gracefully
- Use finalizers for cleanup operations

### Git Operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argoproj-labs/gitops-promoter](https://github.com/argoproj-labs/gitops-promoter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
