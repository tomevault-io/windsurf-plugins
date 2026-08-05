---
trigger: always_on
description: YAML and JSON language provider for Pulumi. Implements a language host gRPC server
---

# Pulumi YAML

YAML and JSON language provider for Pulumi. Implements a language host gRPC server
(`pulumi-language-yaml`) and a converter (`pulumi-converter-yaml`) that translates
YAML/JSON programs into PCL for ejection to other Pulumi languages.

## Start here

| Path                         | What it is                                                              |
|------------------------------|-------------------------------------------------------------------------|
| `pkg/pulumiyaml/run.go`      | Core runtime — evaluates templates, registers resources with the engine |
| `pkg/pulumiyaml/analyser.go` | Schema-aware type analysis of YAML templates                            |
| `pkg/pulumiyaml/ast/`        | AST types for parsed YAML programs                                      |
| `pkg/pulumiyaml/codegen/`    | Code generation - YAML<->PCL conversion                                 |
| `pkg/server/server.go`       | Language host gRPC server implementation                                |
| `pkg/converter/`             | Converter gRPC server implementation                                    |

## Repository Structure

| Directory                    | Contents                                                                       |
|------------------------------|--------------------------------------------------------------------------------|
| `cmd/pulumi-language-yaml/`  | Language host binary entry point                                               |
| `cmd/pulumi-converter-yaml/` | Converter binary entry point                                                   |
| `pkg/pulumiyaml/`            | Core library: AST, syntax, codegen, config, diags, packages                    |
| `pkg/server/`                | Language host gRPC server                                                      |
| `pkg/converter/`             | Converter gRPC server                                                          |
| `pkg/tests/`                 | Integration tests with testdata and transpiled examples                        |
| `examples/`                  | 18 example projects (AWS, Azure, EKS, Kubernetes, etc.) (also run as examples) |
| `scripts/`                   | Schema fetching, coverage, and plugin doc utilities                            |

## Command canon

```
make build                    # Build both binaries to bin/
make lint                     # golangci-lint + copyright check
make test                     # Full test suite (builds first, fetches plugins+schemas)
make test_short               # Fast: skips integration tests
make test_live                # Requires AWS credentials (PULUMI_LIVE_TEST=true)
make get_schemas              # Download provider schemas for tests
make get_plugins              # Install Pulumi provider plugins for tests
make lint-golang              # Lint only the go code
```

## Code Conventions

### Forbidden Patterns

- **Do not edit schema JSON files** in `pkg/pulumiyaml/testing/test/testdata/`. They are downloaded by `scripts/get_schemas.sh` and gitignored (partially). Regenerate with `make get_schemas`.
- **Do not edit files in `pkg/tests/transpiled_examples/`** by hand. They are golden files updated with `PULUMI_ACCEPT=true go test --run=TestGenerateExamples ./...`.
- **Do not use root-level `configuration`** — it is deprecated. Use `config` instead.
- **Copyright headers are required** on all Go files except generated examples. CI checks via `pulumictl copyright`.
- **Do not expose Go types in user-facing errors.** Show user-meaningful types (e.g., "got boolean"), not Go types (e.g., `*ast.BooleanExpr`). Reviewers consistently reject this.
- **Do not use `path.Dir`/`path.Join` for filesystem paths.** Use `filepath.Dir`/`filepath.Join` — `path` is for URL paths and breaks on Windows.
- **Do not maintain manual skip lists for tests.** Use programmatic checks (e.g., `if strings.HasPrefix(name, "provider-") { skip }`) instead of listing test names. Manual lists rot.

### Review Expectations

These patterns are consistently flagged in PR review:

- **Error messages must be precise and non-redundant.** Consolidate duplicate diagnostics into summary/detail. Always return diagnostics — don't swallow them.
- **Naming must match pulumi/pulumi and PCL.** Field names, resource option names, and codegen output must align with the core SDK. Check PCL equivalents before naming new fields.
- **Tests must exercise behavior, not just declare it.** A config test must use the config value, not just declare it exists. Reviewers catch tests that don't actually assert anything meaningful.
- **Changelog entries need precise wording.** Describe what changed from the user's perspective, not internal implementation details.
- **Prefer simple APIs.** Avoid unnecessary boolean parameters, intermediate checks, or complex type hierarchies when a simpler design works. Flip if-guards for fail-fast readability.
- **Dependency upgrades propagate downstream.** Go version bumps and pulumi SDK upgrades affect consumers like docsgen and terraform-bridge. Consider downstream impact.

### Testing

- Use `testify/assert` and `testify/require`. Prefer `assert.Equal` on whole structs over per-field assertions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pulumi/pulumi-yaml](https://github.com/pulumi/pulumi-yaml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
