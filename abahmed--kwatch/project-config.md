---
trigger: always_on
description: Guidance for humans and AI agents making changes to this repository. For user-facing
---

# AGENTS.md — conventions for working on kwatch

Guidance for humans and AI agents making changes to this repository. For user-facing
contribution process, see [CONTRIBUTING](./CONTRIBUTING.md); for how the product behaves,
see [docs/architecture.md](./docs/architecture.md).

## The gate

Every change must pass before you are done:

```sh
go build ./... && go vet ./... && go test ./... && golangci-lint run
```

The repository also enforces formatting and line length. Run the complete gate
with `make verify`; it includes `line-check` and `git diff --check` should be
clean before handoff.

- Linters: errcheck, gocritic, gocyclo, govet, ineffassign, unparam, unused (`.golangci.yml`).
- **Cyclomatic complexity limit is 20** (`gocyclo min-complexity: 20`), tests included. When a
  function exceeds it, extract helpers or table data instead of raising the threshold.
- Formatting: `goimports` with `local-prefixes github.com/abahmed/kwatch` (stdlib first,
  third-party second, kwatch last).
- Test files are exempt from errcheck/unparam/gocritic/gocyclo; `internal/controller` is
  exempt from errcheck (informer wiring intentionally ignores AddEventHandler returns).

## Coding style directives

These directives apply to humans and coding agents. Preserve the existing
architecture unless a change explicitly expands its scope.

- Use `goimports`, not only `gofmt`, with the repository local prefix. Keep
  imports grouped as standard library, third-party dependencies, then kwatch.
- Keep every touched or newly created line at 80 columns or fewer. Wrap calls,
  signatures, composite literals, and comments; do not hide violations by
  disabling the line checker.
- Keep every Go file below 400 lines. When a file grows, split it by
  responsibility and use semantic names such as `conditions.go` or
  `payload_limits_test.go`, not anonymous numeric fragments.
- Match the package name to the final directory component. For example,
  `internal/graphcontext` must declare `package graphcontext`.
- Add short comments only where they explain an invariant, compatibility rule,
  or non-obvious decision. Avoid decorative separator comments and restating
  the code.
- Prefer small functions with one responsibility. Extract `build*`, `parse*`,
  `apply*`, and `validate*` helpers before complexity or readability suffers.
- Return errors to callers. `os.Exit` belongs only in the top-level command
  entrypoint; libraries and subcommands must remain testable.
- Inject clocks, HTTP clients, listers, and other time- or I/O-dependent
  collaborators through constructors or setters. Production code must not
  call `time.Now()` directly when a decision can be tested with a fake clock.
- Keep package globals immutable or narrowly scoped. Use an explicit registry
  or dependency seam for mutable process state; retain compatibility aliases
  only when removing them would break external users.
- Use `metrics.DefaultRegistry()` at internal call sites. Do not introduce new
  direct uses of `metrics.Default` or hidden singleton clients.
- Construct shared Kubernetes or HTTP clients in `internal/app` and pass them
  into monitors, providers, and integrations that need them.
- Import `internal/graphcontext` with an explicit alias when the standard
  library `context` is also in scope; never disguise a package-name mismatch.
- Do not duplicate provider transport or retry logic. Providers build payloads
  and call `alert/util.Send`; shared utilities decide status classification,
  timeout, retry, and rate-limit behavior.
- Preserve persistence formats and public constructor compatibility where
  possible. Prefer optional dependencies or adapters over breaking call sites.

### Naming standard

- Use `New<Type>` for constructors and `Set<Type>` for optional wiring. Keep
  constructor arguments ordered as configuration, required dependencies, then
  optional dependencies.
- Name methods after the domain action: `Process`, `Resolve`, `Snapshot`, and
  `Validate`. Avoid vague verbs such as `Do`, `HandleIt`, or `Create` when the
  resource type is known.
- Use singular package names and lower-case file names. Group files by one
  responsibility: `graph_resources.go`, `group_flush.go`, and
  `payload_limits_test.go` are preferred examples.
- Follow Go initialisms consistently: `ID`, `UID`, `URL`, `HTTP`, `API`, `PVC`,
  and `JSON`. Do not introduce a new spelling variant for an existing public
  identifier; add a compatibility wrapper when a rename is unavoidable.
- Use `camelCase` for local names, `PascalCase` for exported names, and avoid
  redundant package prefixes such as `config.ConfigManager`.
- Use `Test<Type><Behavior>` for tests. Name table cases by behavior, not by
  implementation order or issue number.

### Test and refactor directives

- Before splitting a test file, identify package-level fixtures, helper types,
  and imports. Copy required declarations into the correct focused file and
  run that package's tests immediately after the split.
- Keep tests deterministic: use injected clocks and fake clients rather than
  sleeps, wall-clock assertions, or live network calls.
- Add or update focused tests for every behavior change, especially lifecycle

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abahmed/kwatch](https://github.com/abahmed/kwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
