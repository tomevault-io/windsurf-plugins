---
trigger: always_on
description: GoZod is a TypeScript Zod v4-inspired validation library for Go 1.26+. The root package is a user-facing facade over concrete schemas in `types/`, shared contracts in `core/`, runtime machinery in `internal/`, and JSON Schema translation in `jsonschema/`.
---

# GoZod

GoZod is a TypeScript Zod v4-inspired validation library for Go 1.26+. The root package is a user-facing facade over concrete schemas in `types/`, shared contracts in `core/`, runtime machinery in `internal/`, and JSON Schema translation in `jsonschema/`.

- **Reference implementation:** TypeScript Zod v4 in [`.reference/zod/`](.reference/zod/) when that submodule is initialized.
- **For user-facing examples and installation:** see [README.md](README.md).

## Commands

Run these from the repository root.

```bash
task test                         # Run the default test suite
task test:race                    # Run race-enabled tests for lightweight packages
task lint                         # Run golangci-lint and the tidy check
task golangci-lint                # Run golangci-lint only
task tidy-lint                    # Verify go.mod and go.sum stay tidy
task fmt                          # Run go fmt ./...
task vet                          # Run go vet ./...
task verify                       # Run deps, fmt, vet, lint, test, and govulncheck
go build ./...                    # Verify all packages compile
go test -tags=contractcheck ./types # Audit compile-time schema contracts
```

## Architecture

```text
gozod/
├── gozod_*.go       # Root facade: constructors, aliases, errors, metadata, JSON Schema entry points
├── cmd/gozodgen/    # Code generator for struct-tag schemas
├── coerce/          # z.coerce-style constructors for automatic input conversion
├── core/            # Public contracts, config, issue codes, shared types
├── docs/            # User documentation
├── examples/        # Runnable examples
├── internal/        # Engine, checks, issues, and runtime helpers
├── jsonschema/      # To/From JSON Schema conversion
├── locales/         # Localized validation messages
├── pkg/             # Shared helpers (clone, coercion, maps, regex, reflection, tags, transforms)
├── types/           # Concrete schema implementations and fluent APIs
├── .agents/rules/   # Schema, check, testing, naming, and structure guides
└── .reference/zod/  # Optional TypeScript Zod reference checkout
```

## Agent Workflow

- Read the relevant guides in [`.agents/rules/`](.agents/rules/) before changing schema internals, checks, naming, or tests.
- Verify every user-facing claim against the current repo state. Do not document commands, files, examples, or reference paths you have not checked.
- Initialize [`.reference/zod/`](.reference/zod/) before relying on it for parity work against TypeScript Zod.

## Design Philosophy

- **KISS** — Keep the public surface centered on root constructors, fluent modifiers, and the `Parse` / `StrictParse` pair.
- **Single Responsibility** — `core` defines contracts, `types` own schema behavior, `internal` owns execution, and `jsonschema` owns translation.
- **Open-Closed** — Extend behavior through checks, refinements, metadata, and new schema types instead of special-casing existing ones.
- **APIs as language** — Chains such as `gozod.String().Min(2).Email()` and `gozod.FromStruct[T]()` should read like the validation intent.
- **Beauty is structural** — The root package stays a thin facade while dependency direction remains one-way toward runtime internals.
- **Never:** accidental complexity, feature gravity, abstraction theater, configurability cope.

## API Design Principles

- **Progressive Disclosure**: Start from the root `gozod` package and `FromStruct[T]()`. Drop to `coerce/`, `types/`, `jsonschema/`, or `cmd/gozodgen` only when the simple path stops fitting.

## Coding Rules

### Must Follow

- Go 1.26.2 — use modern language features where they simplify code.
- Follow Google Go Best Practices: <https://google.github.io/go-style/best-practices>
- Follow Google Go Style Decisions: <https://google.github.io/go-style/decisions>
- KISS/DRY/YAGNI — no speculative helpers, no duplicated validation logic, no unused API surface.
- Preserve the two parsing modes: `Parse(any)` for dynamic inputs and `StrictParse(T)` for compile-time constrained inputs.
- Keep modifier methods copy-on-write. `Optional`, `Nilable`, `Default`, `Prefault`, `Describe`, `Meta`, and similar fluent modifiers must return a new configured schema.
- Route parsing through `internal/engine` helpers. Do not reimplement parsing pipelines in root facade files.
- Keep dependency direction one-way. Schema files in `types/` do not import each other; shared behavior belongs in `internal/`, `pkg/`, or `coerce/`.
- Update [types/constraints_verify.go](types/constraints_verify.go) when changing schema interfaces or adding new schema families.
- Keep docs, examples, and task commands aligned with real exports and real files.

### Go 1.26 Features

| Feature | Where Used |
|---------|------------|
| Self-referential generic constraints | [core/constraints.go](core/constraints.go) defines fluent schema contracts |
| `new(expr)` | Pointer construction across parser, schema, locale, and JSON Schema helpers |
| `maps.Clone` | [core/interfaces.go](core/interfaces.go) clones map-backed internals safely |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaptinlin/gozod](https://github.com/kaptinlin/gozod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
