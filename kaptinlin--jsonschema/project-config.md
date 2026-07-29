---
trigger: always_on
description: High-performance JSON Schema Draft 2020-12 validation for Go. The package combines a compiler, a direct validation API for JSON/maps/structs, default-aware unmarshaling, a fluent constructor API, and struct-tag-driven schema generation.
---

# jsonschema

High-performance JSON Schema Draft 2020-12 validation for Go. The package combines a compiler, a direct validation API for JSON/maps/structs, default-aware unmarshaling, a fluent constructor API, and struct-tag-driven schema generation.

For end-user installation and usage examples, see [README.md](README.md) and the guides in [docs/](docs/).

## Commands

Run commands from this directory.

```bash
task help          # Show available task targets
task test          # Run all tests with the race detector
task lint          # Run golangci-lint and go mod tidy checks
task bench         # Run benchmarks
task verify        # Run deps, fmt, vet, lint, test, and govulncheck
```

## Architecture

```text
jsonschema/
├── *.go                    # Core compiler, schema model, validators, constructor API, and unmarshal logic
├── cmd/schemagen/          # Code generator for struct-tag-driven schemas
├── docs/                   # Human-facing guides for API, validation, unmarshal, formats, and tags
├── examples/               # Runnable examples for the major workflows
├── pkg/tagparser/          # Shared struct-tag parsing used by runtime generation and schemagen
├── tests/                  # Integration tests and official JSON Schema suite coverage
└── testdata/               # Test fixtures and external suite data
```

Key entry points:

- `Compiler` in `compiler.go` — compiles schemas, manages references, loaders, formats, and default functions.
- `Schema` in `schema.go` — holds the compiled schema graph and schema metadata.
- Validation methods in `validate.go` — `Validate`, `ValidateJSON`, `ValidateStruct`, `ValidateMap`.
- Unmarshal support in `unmarshal.go` — applies defaults without performing validation.
- Constructor API in `constructor.go` and `keywords.go` — builds schemas directly in Go.
- Struct-tag generation in `struct_tags.go` — generates schemas from Go types at runtime.

## Design Philosophy

- **KISS** — Model JSON Schema keywords directly on `Schema` and keep the public surface centered on compiler, schema, constructor, and struct-tag workflows.
- **DRY** — Keep runtime validation, constructor keywords, struct-tag generation, and `cmd/schemagen` aligned to the same schema semantics instead of inventing parallel rule systems.
- **YAGNI** — Prefer one clear API per workflow over layered convenience wrappers that duplicate behavior.
- **APIs as language** — Constructor helpers should read like schema text: `Object`, `Prop`, `Required`, `If(...).Then(...).Else(...)`.
- **Errors as teachers** — Return structured errors with keyword, field, and location context so callers can diagnose invalid schemas and payloads quickly.
- **Never:** accidental complexity, feature gravity, abstraction theater, configurability cope.

## API Design Principles

- **Progressive Disclosure** — Keep `schema.Validate(...)` convenient, while exposing `ValidateJSON`, `ValidateStruct`, `ValidateMap`, compiler hooks, and constructor APIs for advanced cases.

## Coding Rules

### Must Follow

- Use the Go version declared in `go.mod`; use the modern features already present in this repository when they simplify code.
- Keep validation and unmarshaling separate. `Schema.Unmarshal` applies defaults but must not silently become a validator.
- Keep validation entry points behaviorally aligned. When a change affects validation semantics, add coverage for the relevant combination of `Validate`, `ValidateJSON`, `ValidateStruct`, and `ValidateMap`.
- Preserve JSON Schema Draft 2020-12 semantics. `format` remains annotation-only unless the caller opts in with `Compiler.SetAssertFormat(true)`.
- Keep constructor helpers chainable and close to JSON Schema vocabulary.
- Preserve deterministic generated schema output unless an option explicitly allows otherwise, such as `RequiredSortNone`.
- Reuse the modern stdlib/tooling patterns already in the codebase: `slices`, `maps`, `for range N`, and `testing.B.Loop()`.
- Follow [Google Go Best Practices](https://google.github.io/go-style/best-practices).
- Follow [Google Go Style Decisions](https://google.github.io/go-style/decisions).

### Forbidden

- No `panic` in library code — return errors and wrap with context.
- No silent behavior changes to `required`, `omitempty`, or `omitzero` semantics without targeted tests.
- No duplicate validation logic paths that drift between JSON, map, and struct workflows.
- No premature abstraction — three similar lines are better than a helper used once.
- No feature creep — add only behavior the package needs today.
- No working around dependency bugs — if a dependency is broken, report it instead of reimplementing it inline.

## Dependency Issue Reporting

When you encounter a bug, limitation, or unexpected behavior in a dependency library:

1. Do **not** work around it by reimplementing the dependency's functionality.
2. Do **not** skip the dependency and write a private replacement inline.
3. Create a report file at `reports/<dependency-name>.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaptinlin/jsonschema](https://github.com/kaptinlin/jsonschema) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
