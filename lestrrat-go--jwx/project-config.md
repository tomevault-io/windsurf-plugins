---
trigger: always_on
description: Generates `WithToken(v Token) ParseOption` function.
---

# AGENTS.md

## For Module Consumers

If you are writing code that *uses* jwx (not developing jwx itself):

- **Examples**: See [`github.com/jwx-go/examples`](https://github.com/jwx-go/examples) for runnable usage patterns (locally at `examples/` when checked out via `go.work`)
- **Documentation**: See `docs/` directory and package READMEs
- **API Reference**: Use `go doc` or https://pkg.go.dev/github.com/lestrrat-go/jwx/v4
- **Migrating from v3?** See [`MIGRATION.md`](MIGRATION.md) for a complete guide, and [`github.com/jwx-go/jwxmigrate`](https://github.com/jwx-go/jwxmigrate) for machine-readable migration rules and automated checking

The rest of this document focuses on developing the jwx library itself.

---

## Go Version

This project requires **Go 1.26.0** or later. Check `go.mod` for the exact version.

## GOEXPERIMENT

v4 depends on `encoding/json/v2` which requires `GOEXPERIMENT=jsonv2`. The Makefile exports this automatically, but any direct `go build`, `go test`, or `go run` invocation **must** set it:

```bash
GOEXPERIMENT=jsonv2 go test ./...
GOEXPERIMENT=jsonv2 go build ./...
```

Without this, builds fail with `build constraints exclude all Go files` errors.

## Module Path vs Physical Layout

This repository uses a **flat layout** with vanity import paths. There is no physical `v4/` directory.

| Branch | Module Path | Physical Root |
|--------|-------------|---------------|
| `develop/v4` | `github.com/lestrrat-go/jwx/v4` | `/` (repo root) |

`import "github.com/lestrrat-go/jwx/v4/jwt"` → files are at `./jwt/`, not `./v4/jwt/`.

## Code Generation

### Immutable Rule

**NEVER edit files ending in `_gen.go` directly.** These are generated files. Edit the generator sources instead.

### Generated Files Pattern

Files matching `*_gen.go` are generated. Examples:
- `jwt/options_gen.go`
- `jwt/token_gen.go`
- `jws/headers_gen.go`
- `jwk/rsa_gen.go`
- `jwa/signature_gen.go`

### Generator Locations

All generators live in a single binary at `internal/jwxcodegen/cmd/jwxcodegen/`. Each is invoked via subcommand through `scripts/jwxcodegen.sh`, which builds and caches the binary. The `//go:generate` directives in each package call `scripts/jwxcodegen.sh` directly.

| Generator | Subcommand | Input Files | Output |
|-----------|------------|-------------|--------|
| `genoptions` | `generate-options` | `{jwa,jwe,jwk,jws,jwt}/options.yaml` | `*/options_gen.go` |
| `genjwt` | `generate-jwt` | `jwt/objects.yml` | `jwt/*_gen.go` |
| `genheaders` | `generate-headers` | `jws/objects.yml` | `jws/headers_gen.go` |
| `genheaders` | `generate-headers` | `jwe/objects.yml` | `jwe/headers_gen.go` |
| `genjwk` | `generate-jwk` | `jwk/objects.yml` | `jwk/*_gen.go` |
| `genjwa` | `generate-jwa` | `jwa/objects.yml` | `jwa/*_gen.go` |
| `genreadfile` | `generate-readfile` | - | ReadFile helpers |

### Regeneration Commands

```bash
# Regenerate all code (includes options via `go generate .`)
make generate

# Regenerate specific package (objects/types only, NOT options)
make generate-jwt
make generate-jws
make generate-jwe
make generate-jwk
make generate-jwa

# Regenerate options only (options.yaml → options_gen.go for all packages)
go generate .
# or directly:
./scripts/jwxcodegen.sh generate-all-options
```

**Important:** `make generate-<pkg>` does **not** regenerate options. If you
edit an `options.yaml` file, run `make generate` or `go generate .`.

## Functional Options Pattern

Options are defined in `{package}/options.yaml` and generated into `{package}/options_gen.go`.

Example `options.yaml` entry:

```yaml
options:
  - ident: Token
    interface: ParseOption
    argument_type: Token
    comment: |
      WithToken specifies the token instance...
```

Generates `WithToken(v Token) ParseOption` function.

## Multi-Module Structure

This repository contains multiple Go modules. The nested modules use `replace` directives for local development.

| Module | Path | Purpose |
|--------|------|---------|
| Main | `./go.mod` | Core library |
| Examples | [`github.com/jwx-go/examples`](https://github.com/jwx-go/examples) | Usage examples (external repo, local at `./examples/` via `go.work`) |
| CLI | `./cmd/jwx/go.mod` | Command-line tool |
| Generators | `./internal/jwxcodegen/go.mod` | Code generators |

Benchmarks live in a separate repository: [github.com/jwx-go/benchmarks](https://github.com/jwx-go/benchmarks).

### Local Development

No `go.work` file is committed. When working across modules (including examples and extensions), create a temporary `go.work` file (it is .gitignored). If a `go.work` file is present, read it to discover local paths to companion modules (extensions, examples, etc.).

## Development Commands

```bash
# Run all tests
make test

# Run short/smoke tests
make smoke

# Generate coverage report
make cover
make viewcover

# Lint
make lint

# Format and tidy
make imports
make tidy
```

### Test Script Details

Tests are run via `./scripts/test.sh` which iterates over:
- `.` (main module)
- `./cmd/jwx`

## Package Directory Map

| Package | Responsibility |
|---------|----------------|
| `jwa/` | Algorithm identifiers (e.g., `RS256`, `ES384`, `A128GCM`) |
| `jwk/` | JSON Web Keys - key representation and management |
| `jws/` | JSON Web Signatures - `Sign()` and `Verify()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lestrrat-go/jwx](https://github.com/lestrrat-go/jwx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
