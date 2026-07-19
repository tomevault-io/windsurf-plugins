---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

txeh is a Go library and CLI utility for managing /etc/hosts file entries. It provides programmatic and command-line access to add, remove, and query hostname-to-IP mappings. Originally built to support [kubefwd](https://github.com/txn2/kubefwd) for Kubernetes port-forwarding.

**Key Features:**
- Thread-safe operations (mutex-protected for concurrent use)
- IPv4 and IPv6 support
- CIDR range operations for bulk add/remove
- Inline comment support for tracking entry sources
- Preserves comments and file formatting
- Cross-platform (Linux, macOS, Windows)

## Quick Reference

```bash
# Full verification (run before every commit)
make verify          # fmt, lint, test-unit, security, coverage-check, go mod verify

# Extended verification (adds dead-code analysis)
make verify-full     # fmt, lint, test, security, coverage-check, dead-code, go mod verify

# Format, lint, test individually
make fmt             # gofmt + goimports
make lint            # golangci-lint (37 linters, no test relaxations)
make test-unit       # go test -race ./...
make test-short      # Fast tests (no race detection)
make coverage        # Coverage report (HTML + summary)
make coverage-check  # Enforce 80% coverage threshold
make security        # gosec + govulncheck
make mutate          # Mutation testing (gremlins, 60% efficacy threshold)
```

## Architecture

```
┌─────────────────┐
│   txeh/txeh.go  │  CLI entry point (Cobra-based)
│    (cmd/)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│    txeh.go      │  Core library (public API)
│   (package)     │
└─────────────────┘
         │
         ▼
┌─────────────────┐
│  /etc/hosts     │  System hosts file
└─────────────────┘
```

### Key Components

1. **Library** (`txeh.go`): Core functionality for parsing, modifying, and rendering hosts files
   - `Hosts` struct: Thread-safe wrapper with mutex protection
   - `HostFileLine`: Represents a single line (ADDRESS, COMMENT, EMPTY, UNKNOWN)
   - `HostsConfig`: Configuration for read/write paths or raw text input

2. **CLI** (`txeh/` directory): Cobra-based command-line interface
   - Entry point: `txeh/txeh.go`
   - Commands in `txeh/cmd/`: add, remove, list, show, version
   - Global flags: `--dryrun`, `--quiet`, `--read`, `--write`, `--max-hosts-per-line`

## Code Standards

### Style
- Follow [Google Go Style Guide](https://google.github.io/styleguide/go/)
- All code must pass `golangci-lint` with project config
- Maximum cyclomatic complexity: 10 per function
- Maximum cognitive complexity: 15 per function
- Maximum function arguments: 5

### Documentation
- All exported functions, types, and packages require doc comments
- Doc comments must start with the name being documented
- Comments should end with a period

### Error Handling
- Always check and handle errors (no `_` for errors)
- Wrap errors at boundaries: `fmt.Errorf("context: %w", err)`
- Use `errors.Is` and `errors.As`, never string comparison
- Define sentinel errors for expected conditions

```go
// Example error handling pattern
func (h *Hosts) Save() error {
    if err := h.writeFile(); err != nil {
        return fmt.Errorf("save hosts file: %w", err)
    }
    return nil
}
```

### Testing
- Test coverage minimum: 80% (CI enforced)
- Use table-driven tests for multiple cases
- Use `t.Parallel()` for independent tests
- Race detection required: `go test -race ./...`
- Property-based tests with `rapid` for invariant verification

### Verification Stack

All changes go through a 4-level verification process (see [AI-Verified Development](https://imti.co/ai-verified-development/)):

1. **Static analysis** - `make lint` runs golangci-lint with 37 linters. Runs in seconds, catches type errors, complexity violations, security anti-patterns.
2. **Unit tests** - `make test-unit` runs with `-race`. Coverage gate at 80%. `make coverage-check` enforces this locally.
3. **Integration/E2E tests** - Tagged test suites (`-tags=integration`, `-tags=e2e`) for system-level validation.
4. **Mutation testing** - `make mutate` runs gremlins with a 60% efficacy threshold. Validates that tests catch real bugs, not just exercise code.

Do not review AI code until every level passes. `make verify` runs levels 1-2 plus security. `make verify-full` adds dead-code analysis.

### Acceptance Criteria

Define specs as Given/When/Then assertions before writing code. Each "then" becomes a concrete test with a hardcoded expected value.

```
Given a hosts file with "127.0.0.1 myhost"
When RemoveHost("myhost") is called
Then ListHostsByIP("127.0.0.1") returns an empty list
```

Write the test first, confirm it fails, then implement.

### AI-Specific Rules

- **No tautological tests.** Tests must encode specific expected outputs as hardcoded values. Never reimplement function logic inside assertions.
- **No hallucinated imports.** Verify every dependency exists and is actively maintained before adding it.
- **Human review required.** A human must review and approve every line of code before commit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eko/monday](https://github.com/eko/monday) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
