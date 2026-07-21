---
trigger: always_on
description: Generates comprehensive `.testsuite.yaml` integration test files for OpenShift API type definitions:
---

This file provides guidance to AI agents when working with code in this repository.

This is the OpenShift API repository - the canonical location of OpenShift API type definitions and serialization code. It contains:

- API type definitions for OpenShift-specific resources (Custom Resource Definitions)
- FeatureGate management system for controlling API availability across cluster profiles
- Generated CRD manifests and validation schemas
- Integration test suite for API validation

## Key Architecture Components

### FeatureGate System
The FeatureGate system (`features/features.go`) controls API availability across different cluster profiles (Hypershift, SelfManaged) and feature sets (Default, TechPreview, DevPreview). Each API feature is gated behind a FeatureGate that can be enabled/disabled per cluster profile and feature set.

### API Structure
APIs are organized by group and version (e.g., `route/v1`, `config/v1`). Each API group contains:
- `types.go` - Go type definitions
- `zz_generated.*` files - Generated code (deepcopy, CRDs, etc.)
- `tests/` directories - Integration test definitions
- CRD manifest files

## Common Development Commands

### Building
```bash
make build              # Build render and write-available-featuresets binaries
make clean              # Clean build artifacts
```

### Code Generation
```bash
make update             # Alias for update-codegen-crds
```

#### Targeted Code Generation
When working on a specific API group/version, you can regenerate only the affected CRDs instead of all CRDs:

```bash
# Regenerate CRDs for a specific API group/version
make update-codegen API_GROUP_VERSIONS=operator.openshift.io/v1alpha1
make update-codegen API_GROUP_VERSIONS=config.openshift.io/v1
make update-codegen API_GROUP_VERSIONS=route.openshift.io/v1

# Multiple API groups can be specified with comma separation
make update-codegen API_GROUP_VERSIONS=operator.openshift.io/v1alpha1,config.openshift.io/v1
```

**Important:** While using `API_GROUP_VERSIONS` is faster for iteration (e.g., when developing tests),
it generates invalid OpenAPI data. This targeted generation is useful during development cycles, but you
**must run `make update`** (without `API_GROUP_VERSIONS`) to regenerate all files correctly before
committing changes. The full `make update` ensures all generated files, including OpenAPI schemas, are
properly synchronized.

**Workflow:**
- During iteration: `make update-codegen API_GROUP_VERSIONS=your.group/v1` (fast feedback)
- Before committing: `make update` (ensures correctness)

### Testing
```bash
make test-unit          # Run unit tests
make integration        # Run integration tests (in tests/ directory)
go test -v ./...        # Run tests for specific packages

# Run integration tests for specific API groups
make -C config/v1 test  # Run tests for config/v1 API group
make -C route/v1 test   # Run tests for route/v1 API group
make -C operator/v1 test # Run tests for operator/v1 API group
```

### Validation and Verification
```bash
make verify             # Run all verification checks
make verify-scripts     # Verify generated code is up to date
make verify-codegen-crds # Verify CRD generation is current
make lint               # Run golangci-lint (only on changes from master)
make lint-fix           # Auto-fix linting issues where possible
```

## Adding New APIs

All APIs should start as tech preview.
New fields on stable APIs should be introduced behind a feature gate `+openshift:enable:FeatureGate=MyFeatureGate`.


### For New Stable APIs (v1)
1. Create the API type with proper kubebuilder annotations
2. Include required markers like `+openshift:compatibility-gen:level=1`
3. Add validation tests in `<group>/<version>/tests/<crd-name>/`
4. Run `make update-codegen-crds` to generate CRDs

### For New TechPreview APIs (v1alpha1)
1. First add a FeatureGate in `features/features.go`
2. Create the API type with `+openshift:enable:FeatureGate=MyFeatureGate`
3. Add corresponding test files
4. Run generation commands

### Adding FeatureGates
Add to `features/features.go` using the builder pattern:
```go
FeatureGateMyFeatureName = newFeatureGate("MyFeatureName").
    reportProblemsToJiraComponent("my-jira-component").
    contactPerson("my-team-lead").
    productScope(ocpSpecific).
    enableIn(configv1.TechPreviewNoUpgrade).
    mustRegister()
```

## Testing Framework

The repository includes a comprehensive integration test suite in `tests/`. Test suites are defined in `*.testsuite.yaml` files alongside API definitions and support:
- `onCreate` tests for validation during resource creation
- `onUpdate` tests for update-specific validations and immutability
- Status subresource testing
- Validation ratcheting tests using `initialCRDPatches`

Use `tests/hack/gen-minimal-test.sh $FOLDER $VERSION` to generate test suite templates.

## Container-based Development
```bash
make verify-with-container    # Run verification in container
make generate-with-container  # Run code generation in container
```

Uses `podman` by default, set `RUNTIME=docker` or `USE_DOCKER=1` to use Docker instead.

## Custom Claude Code Commands

### Generate Tests
```
/generate-tests <path-to-types-file-or-api-directory>
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openshift/machine-config-operator](https://github.com/openshift/machine-config-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
