---
trigger: always_on
description: This file provides essential context and instructions for AI coding agents working on the vArmor codebase. It complements the README by offering build steps, conventions, and security constraints that agents need to operate correctly.
---

# vArmor Agent Development Guide

This file provides essential context and instructions for AI coding agents working on the vArmor codebase. It complements the README by offering build steps, conventions, and security constraints that agents need to operate correctly.

## Setup Commands

### Building

- `make build` - Full build (manifests, codegen, Mozilla CA bundle, eBPF, binary)
- `make local` - Build local binary only (fast path for Go-only changes)
- `make manifests` - Generate CRD manifests via controller-gen
- `make generate` - Generate DeepCopy and other boilerplate code
- `make build-ebpf` - Compile eBPF programs
- `make copy-ebpf` - Copy compiled eBPF artifacts to pkg directories
- `make update-mozilla-bundle` - Refresh embedded Mozilla CA bundle (set `SKIP_MOZILLA_BUNDLE_UPDATE=1` for offline builds)
- `make docker-build-proxyinit` - Build proxyinit image (amd64/arm64)
- `make sync-proxy-images` - Build proxyinit + sync Envoy images to registry

### Testing

- `make test` - Full test suite (manifests + generate + fmt + vet + unit tests)
- `make test-unit` - Unit tests with coverage profile
- `go test ./path/to/package -run TestFunctionName` - Run single test
- `go test ./path/to/package` - Run all tests in package
- `go test -v ./path/to/package` - Verbose output

### Linting

- `make fmt` - Format code (`go fmt` + `goimports -w ./`)
- `make vet` - Static analysis (`go vet ./...`)

### Development Workflow

1. `make build` (if CRDs/eBPF/CA changed) or `make local` (Go-only)
2. `./scripts/deploy_resources.sh test` - Install CRDs and resources
3. `sudo ./bin/vArmor -kubeconfig=./varmor-manager.kubeconfig -v 3` - Run manager
4. `sudo ./bin/vArmor -agent -kubeconfig=./varmor-agent.kubeconfig -v 3` - Run agent

## Code Style

### Licensing

- All source files: Apache 2.0 header, `// Copyright 202X vArmor Authors`
- Package comments: `// Package <name> implements...`

### Imports

Three groups separated by blank lines, alphabetically sorted within each:
1. Standard library
2. Third-party packages
3. Internal packages (use descriptive aliases: `varmor "github.com/bytedance/vArmor/apis/varmor/v1beta1"`)

### Naming

| Scope | Convention | Example |
|-------|-----------|---------|
| Packages | lowercase, single word | `policy`, `audit` |
| Exported types | PascalCase | `PolicyController` |
| Exported functions | PascalCase | `ValidateAddPolicy` |
| Unexported | camelCase | `syncHandler` |
| Constants (exported) | PascalCase | `MaxRetries` |
| Variables | camelCase, descriptive | `kubeClient`, `vpInformer` |

### Error Handling

- Always return errors; never ignore
- Wrap with context: `fmt.Errorf("context: %w", err)`
- Check nil before pointer dereference
- Use `k8errors.IsNotFound(err)` for K8s errors

### Logging

- `logr.Logger` for structured logging (pass as struct field)
- Levels: `log.Info()`, `log.Error()`, `log.V(3).Info()` for debug
- `zerolog.Logger` for audit events (in `pkg/auditor/`)

## Testing Instructions

- Use `github.com/stretchr/testify/assert` for assertions
- Naming: `Test<Function>_<Scenario>` (e.g., `TestValidateAddPolicy_ValidVarmorPolicy`)
- Use table-driven tests with `t.Run()` for subtests
- Test both success and failure paths
- Mock K8s clients via `pkg/client/clientset/versioned/fake`
- **Always run `make test` before committing. All tests must pass.**

## Security Considerations

**This section is critical. Violations here can introduce exploitable vulnerabilities.**

### YAML Injection Prevention (NetworkProxy Renderer)

The NetworkProxy profile renderer (`internal/networkproxy/profile/renderer.go`) generates Envoy xDS YAML by string interpolation. **ALL user-controlled values embedded in double-quoted YAML scalars MUST be escaped with `yamlEscapeScalar()`.** Never use raw `fmt.Sprintf` for user input.

```go
// WRONG - YAML injection vulnerability
sb.WriteString(fmt.Sprintf(`    exact: "%s"
`, userValue))

// CORRECT - escaped against injection
sb.WriteString(fmt.Sprintf(`    exact: "%s"
`, yamlEscapeScalar(userValue)))
```

**Dangerous characters** (rejected at webhook, escaped at renderer):
- C0 controls: U+0000-U+001F (includes NUL, TAB, LF, CR)
- DEL: U+007F
- YAML structural: `\` (backslash), `"` (double quote)
- YAML 1.1 line breaks: U+0085 (NEL), U+2028 (LS), U+2029 (PS)

**Safe characters** (pass through unmodified):
- All printable ASCII (U+0020-U+007E) except `\` and `"`
- This includes: `*` `.` `-` `_` `:` `/` `[` `]` `%` `?` `&` `=` `~` `+` `@` `!` space

**Defense-in-depth architecture:**
1. **L1 Webhook** (`internal/policy/validate.go`): `containsYAMLUnsafeChars()` rejects at admission
2. **L3 Renderer** (`yaml_escape.go`): `yamlEscapeScalar()` escapes as safety net

When adding new user-controlled fields to the renderer:
1. Add `yamlEscapeScalar()` at the interpolation point
2. Add `containsYAMLUnsafeChars()` validation in `ValidateNetworkProxyEgress()` or `validateMITMConfig()`
3. Add test cases in both `yaml_escape_test.go` and `validate_test.go`

### Input Validation Rules

- CRD string fields that flow into Envoy config must be validated at webhook admission

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bytedance/vArmor](https://github.com/bytedance/vArmor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
