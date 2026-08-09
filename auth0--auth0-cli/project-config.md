---
trigger: always_on
description: This document provides context and guidelines for AI coding assistants working with the auth0-cli codebase.
---

# AI Agent Guidelines for auth0-cli

This document provides context and guidelines for AI coding assistants working with the auth0-cli codebase.

## Your Role

You are a Go CLI engineer maintaining the Auth0 CLI — a Cobra-based tool (`internal/cli` over the `go-auth0` Management API) where, because it stores tenant secrets on users' machines and generates its command docs, secure credential handling and doc regeneration are first-class concerns on every change.

---

## Working Principles

Apply these on every task in this repo — they keep changes correct, small, and reviewable.

- **Think before coding.** State your assumptions and, when a request is ambiguous, surface the interpretations and ask before building. Recommend a simpler approach when you see one. A clarifying question up front beats a wrong implementation.
- **Simplicity first.** Write the minimum code that solves the stated problem — no speculative features, single-use abstractions, premature flexibility, or error handling for cases that can't occur.
- **Surgical changes.** Touch only what the request requires. Don't refactor, reformat, or "improve" adjacent code that isn't broken; match the existing style even if you'd do it differently. Every changed line should trace directly to the request. Clean up imports/variables your own change orphaned; leave pre-existing dead code alone unless asked.
- **Goal-driven execution.** Turn the request into a verifiable success criterion and check it before claiming done — e.g. "add a flag" becomes "add the flag, wire it through, add a table-driven test, and regenerate docs." Don't report success you haven't verified.

---

## Project Overview

**auth0-cli** is the official command-line interface for Auth0 — build, manage, and test Auth0 integrations from the terminal.

- **Language:** Go 1.25.8
- **Tech Stack:** Cobra (commands) + pflag, go-auth0 Management SDK (v1 `management` and v2), Sentry crash reporting, zalando/go-keyring for secret storage, terraform-exec (Terraform export), charmbracelet/glamour (markdown rendering)
- **Package Manager:** Go modules — **vendored** (`vendor/` is committed; run `go mod tidy && go mod vendor` after dependency changes)
- **Minimum Platform Version:** Go 1.25.8 (from `go.mod`)
- **Dependencies:** go-auth0 v1.44.0 + v2.14.0, spf13/cobra 1.10.2, getsentry/sentry-go 0.47.0, zalando/go-keyring 0.2.8 · test: stretchr/testify 1.11.1, golang/mock (gomock) 1.6.0

---

## Project Structure

```
auth0-cli/
├── cmd/
│   ├── auth0/            # Main entrypoint — calls cli.Execute()
│   └── doc-gen/          # Generates docs/*.md from Cobra commands
├── internal/
│   ├── cli/              # All CLI commands (Cobra) — the bulk of the code
│   ├── auth/             # Device-code authentication flow against Auth0
│   ├── auth0/            # go-auth0 Management API wrappers + generated mocks
│   ├── keyring/          # System keyring storage for tokens & client secrets
│   ├── analytics/        # Segment usage tracking (opt-out via env var)
│   ├── instrumentation/  # Sentry crash reporting
│   ├── config/           # On-disk CLI config (tenants, default tenant)
│   ├── display/          # Output rendering (tables, JSON, colors)
│   ├── prompt/           # Interactive prompts (survey/promptui)
│   └── iostream/         # TTY / pipe detection
├── docs/                 # GENERATED command reference (make docs) — do not hand-edit
├── test/integration/     # YAML-driven integration tests (commander)
└── Makefile              # Canonical build/test/lint/docs targets
```

### Key Files

| File | Purpose |
|------|---------|
| `cmd/auth0/main.go` | Entry point — thin wrapper over `cli.Execute()` |
| `internal/cli/root.go` | Root command, DI wiring (`cli` struct, renderer, tracker) |
| `internal/cli/cli.go` | `cli` struct, tenant/config setup, API client init |
| `internal/auth/auth.go` | Device-code OAuth flow, token exchange |
| `internal/keyring/keyring.go` | Secret storage abstraction over go-keyring |
| `Makefile` | All build/test/lint/docs commands |

---

## Boundaries

### ✅ Always Do

- Run `make lint` and `make test-unit` before committing.
- Follow the existing Cobra command patterns and naming (see [references/code-style.md](references/code-style.md)).
- Add table-driven unit tests for new functionality; regenerate mocks with `make test-mocks` when an interface changes.
- **Regenerate command docs with `make docs` whenever you add/change a command, flag, or help text.** CI runs `make check-docs` and fails if `docs/` is out of sync.
- Update `README.md` in the same PR when a change touches what it documents — installation, config/auth, the top-level command list, deprecations, or supported workflows (per-flag and per-command detail lives in the generated `docs/`, via `make docs`, not the README). Update `CUSTOMIZATION_GUIDE.md` for Universal Login/branding changes and `MIGRATION_GUIDE.md` for breaking changes.
- After changing dependencies, run `go mod tidy && go mod vendor` — the `vendor/` directory is committed and must stay in sync.
- Route new usage tracking through the existing `analytics.Tracker` (`internal/analytics`) and preserve the `AUTH0_CLI_ANALYTICS=false` opt-out; do not hand-roll a new tracking client.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [auth0/auth0-cli](https://github.com/auth0/auth0-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
