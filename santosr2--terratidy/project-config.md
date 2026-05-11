---
trigger: always_on
description: Single-binary Terraform/Terragrunt quality platform. Go 1.25+ (dev: 1.26), library-first, extensible plugin system.
---

# TerraTidy

Single-binary Terraform/Terragrunt quality platform. Go 1.25+ (dev: 1.26), library-first, extensible plugin system.

## Architecture

```text
cmd/terratidy/          # CLI (Cobra)
internal/
  annotations/          # Suppression annotation parsing and filtering
  buildinfo/            # Build information and versioning
  cache/                # Caching layer
  config/               # YAML config with imports, profiles, glob patterns
  engines/              # format, style, lint, policy
  lsp/                  # Language Server Protocol implementation
  output/               # Text, JSON, SARIF, JUnit, Markdown, HTML, Table, GitHub Actions formatters
  plugins/              # Go (.so), YAML, Bash rule loader
  runner/               # Orchestration, parallel file processing
  vcs/                  # Git integration (--changed flag)
pkg/
  sdk/                  # Public SDK for rule authors
.github/workflows/      # CI/CD pipelines (test, release, security, docs)
assets/                 # Brand assets, icons, logos
docs/site/              # MkDocs documentation site
examples/               # Example configs and custom rules
Formula/                # Homebrew formula (auto-generated)
vscode/                 # VS Code extension (TypeScript, Bun)
tools/scripts/          # Development scripts
Dockerfile              # Container image definition
action.yml              # GitHub Action definition
```

## Core Interfaces

```go
// Every engine implements this (pkg/sdk/types.go)
type Engine interface {
    Name() string
    Run(ctx context.Context, files []string) ([]sdk.Finding, error)
}

// Every rule implements this (pkg/sdk/types.go)
type Rule interface {
    Name() string
    Description() string
    Check(ctx *sdk.Context, file *hcl.File) ([]sdk.Finding, error)
}

// Rules that support auto-fixing also implement Fixer
type Fixer interface {
    Fix(ctx *sdk.Context, file *hcl.File) ([]byte, error)
}
```

## Non-Negotiable Rules

- **Library-first**: Use Go libraries (hclwrite, OPA SDK) where possible. TFLint is invoked as a CLI subprocess. Never `exec.Command("terraform", ...)`.
- **No panic**: Return errors with context: `fmt.Errorf("loading config: %w", err)`
- **No global state**: Pass config through context or parameters
- **No circular deps**: `internal/` is private, `pkg/` is public API
- **Actionable errors**: Include file paths, line numbers, suggestions for the user

## Key Libraries

| Library | Purpose |
| --- | --- |
| `github.com/hashicorp/hcl/v2` | HCL parse and write |
| `github.com/hashicorp/hcl/v2/hclwrite` | AST-based formatting |
| `github.com/open-policy-agent/opa` | Policy engine |
| `github.com/spf13/cobra` | CLI framework |
| `github.com/fsnotify/fsnotify` | File watching |
| `golang.org/x/text` | Text processing |
| `gopkg.in/yaml.v3` | YAML config parsing |
| `github.com/stretchr/testify` | Test assertions |

## Config System

```yaml
# .terratidy.yaml
version: 1

engines:
  fmt:
    enabled: true
    check: false   # Dry-run mode (don't modify files)
    diff: false    # Show diff of changes
  style:
    enabled: true
    fix: false     # Auto-fix style issues
    diff: false
  lint:
    enabled: true
    use_tflint: false        # Enable TFLint integration
    fallback_builtin: true   # Use built-in rules if TFLint unavailable
  policy:
    enabled: false           # Opt-in for policy checking
    policy_dirs:
      - ./policies

# Global settings
severity_threshold: warning  # info|warning|error
fail_fast: false
parallel: true
recursive: true              # Recursive directory traversal (default: true)

# Caching settings
cache:
  disabled: false
  max_age: "5m"
  max_size: 1000

# Output settings
output:
  absolute_paths: false      # Use absolute paths in findings

imports:
  - .terratidy/*.yaml        # Glob patterns for modular configs

exclude:
  - "**/*.generated.tf"      # Glob patterns for files/dirs to exclude
  - "vendor/**"

profiles:
  production:
    description: "Production checks with policy enforcement"
    inherits: ""             # Optional: inherit from another profile
    engines:
      policy:
        enabled: true

plugins:
  enabled: true
  verify_integrity: true     # Verify plugin integrity (default: true)
  directories:
    - .terratidy/plugins
    - ~/.terratidy/plugins
  tags: []                   # Filter plugins by tag (empty = all)
```

## Development

```bash
# Go development
mise install              # Install Go 1.26 + tools
mise run setup            # Install dependencies
mise run build            # Build binary
mise run test             # Unit tests
mise run test:integration # Integration tests
mise run lint             # golangci-lint
mise run check            # fmt + vet + lint + test (run before PR)
mise run build && ./bin/terratidy init-rule --name x --type go|rego|yaml  # Scaffold new rule

# VSCode extension development
cd vscode && bun install     # Install extension deps
cd vscode && bun run compile # Build extension
cd vscode && bun run test    # Run extension tests
cd vscode && bun run lint    # Biome lint/format check
```

## Code Quality

All static analysis runs through `golangci-lint` (`.golangci.yml`):


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [santosr2/TerraTidy](https://github.com/santosr2/TerraTidy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
