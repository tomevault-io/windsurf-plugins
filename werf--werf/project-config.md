---
trigger: always_on
description: All rules in this document are requirements — not suggestions. ALWAYS follow them.
---

# AGENTS

All rules in this document are requirements — not suggestions. ALWAYS follow them.

werf is a CNCF Sandbox CLI tool to implement full-cycle CI/CD to Kubernetes. werf integrates into your CI system and leverages familiar and reliable technologies, such as Git, Dockerfile, Helm, and Buildah. werf uses [werf/nelm](https://github.com/werf/nelm) as its deployment engine.

## Highest-priority rule (MANDATORY)

- NEVER add comments unless they document a non-obvious public API or explain genuinely non-obvious logic. NEVER add comments that restate what the code does, repeat the field/function name, describe obvious error handling, or act as section separators. When in doubt, don't comment.
- ALWAYS use `task` commands for build/test/lint/format — NEVER raw `go build`, `go test`, `go vet`, `go fmt`, or `golangci-lint` directly.
- ALWAYS verify, don't assume — check the actual state before making changes.
- ALWAYS start with the simplest possible solution. If it works, stop. Add complexity only when justified by a concrete, current requirement — NEVER for hypothetical future needs.
- NEVER leave TODOs, stubs, or partial implementations.
- ALWAYS stay within the scope of what was asked. When asked to update a plan — only update the plan, don't change code. When asked to brainstorm/discuss — only discuss, don't write code. When asked to do X — do X and nothing else. NEVER make unsolicited changes.
- NEVER modify CHANGELOG.md, release notes, or other generated/workflow-managed files unless the user explicitly requests it.
- When deleting a block from structured data files (YAML, JSON, TOML), ALWAYS read surrounding lines to verify adjacent content (anchors, references, unrelated entries) is preserved.
- When removing content, ALWAYS clean up orphaned structural elements (comment separators, section headers, blank-line groups) that no longer serve a purpose.
- When renaming a type, function, or constant, ALWAYS rename all related local variables, parameters, and error messages that reference the old name. A rename is not complete until grep for the old name returns zero hits in affected packages.
- When removing a feature that has documentation in multiple languages (e.g. `pages_en/`, `pages_ru/`), ALWAYS apply the same removal to ALL language versions. NEVER assume English-only cleanup is sufficient.

## Code style

### Design (MANDATORY)

> The code style rules below are adapted from [CODESTYLE.md](CODESTYLE.md). If you are asked to update code style rules, update CODESTYLE.md first, then regenerate this section to match, using ALWAYS/NEVER/MUST phrasing.

- ALWAYS prefer stupid and simple over abstract and extendable.
- ALWAYS prefer a bit of duplication over complex abstractions.
- ALWAYS prefer clarity over brevity in names.
- ALWAYS minimize interfaces, generics, embedding.
- ALWAYS prefer fewer types. Prefer no types over few. Prefer data types over types with behavior.
- ALWAYS prefer functions over methods. ALWAYS prefer public fields over getters/setters.
- ALWAYS keep everything private/internal as much as possible.
- ALWAYS validate early, validate a lot. ALWAYS keep APIs stupid and minimal.
- NEVER prefer global state. ALWAYS prefer simplicity over micro-optimizations.
- ALWAYS use libraries for complex things instead of reinventing the wheel.
- NEVER add comments unless they document a non-obvious public API or explain genuinely non-obvious logic. NEVER add obvious/redundant comments, NEVER add comments restating what code does. When in doubt, don't comment.

### Conventions (MANDATORY)

> The code style rules below are adapted from [CODESTYLE.md](CODESTYLE.md). If you are asked to update code style rules, update CODESTYLE.md first, then regenerate this section to match, using ALWAYS/NEVER/MUST phrasing.

- All public functions/methods MUST accept `context.Context` as the first parameter.
- All arguments of a public function are required — passing nil not allowed.
- Optional arguments via `<FunctionName>Options` as the last argument. NEVER use functional options.
- Use guard clauses and early returns to keep the happy path unindented.
- Use `samber/lo` helpers: `lo.Filter`, `lo.Find`, `lo.Map`, `lo.Contains`, `lo.Ternary`, `lo.ToPtr`, `lo.Must`, etc.
- Constructors: `New<TypeName>[...]()`. No network/filesystem calls in constructors.
- Interfaces: ALWAYS add `var _ Animal = (*Dog)(nil)` compile-time check.
- Constants: avoid `iota`. Prefix enum constants with type name: `LogLevelDebug LogLevel = "debug"`.
- Errors: ALWAYS wrap with context: `fmt.Errorf("read config: %w", err)`. Describe what is being done, not what failed. Panic on programmer errors. Prefer one-line `if err := ...; err != nil`.

### Go standard guidelines (MANDATORY)

Follow [Effective Go](https://go.dev/doc/effective_go) and [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments). Commonly violated rules:

- NEVER use `this`/`self` as receiver names. Use 1-2 letter names, consistent across methods.
- NEVER discard errors with `_`. Indent error flow, not happy path.
- NEVER use dot imports.
- NEVER use named returns or naked returns.

## Commands (MANDATORY)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [werf/werf](https://github.com/werf/werf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
