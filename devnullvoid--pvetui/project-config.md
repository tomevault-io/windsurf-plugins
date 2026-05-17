---
trigger: always_on
description: **Last Updated:** March 2026 (CLI subcommands) | **For:** pvetui - Proxmox TUI
---

# AGENT INSTRUCTIONS

**Last Updated:** March 2026 (CLI subcommands) | **For:** pvetui - Proxmox TUI

## Table of Contents
- [Initial Setup](#initial-setup)
- [Development Workflow](#development-workflow)
- [Quick Reference](#quick-reference)
- [Code Quality Standards](#code-quality-standards)
- [Style Guidelines](#style-guidelines)
- [Documentation Requirements](#documentation-requirements)
- [Commit Standards](#commit-standards)
- [Security and Performance](#security-and-performance)
- [Tools and Environment](#tools-and-environment)
- [Testing Strategy](#testing-strategy)
- [Project Context](#project-context)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Recent Code Quality Improvements](#recent-code-quality-improvements)
  - [Key Files](#key-files)
  - [Authentication](#authentication)
  - [Caching Strategy](#caching-strategy)
  - [Testing](#testing)
  - [Plugin Architecture](#plugin-architecture)
  - [Plugin Development Guidelines](#plugin-development-guidelines)
  - [Architectural Decision Log](#architectural-decision-log)
- [Common Pitfalls](#common-pitfalls)
- [Troubleshooting](#troubleshooting)
- [Maintaining This Document](#maintaining-this-document)

---

The following conventions must be followed for any changes in this repository.

## Initial Setup

1. Run development setup: `make dev-setup` (installs required tools and validates environment).
2. The embedded noVNC client lives in a git subtree; sync it with upstream when needed via `make update-novnc`.
3. For enhanced development experience (optional):
   - Install direnv: `sudo pacman -S direnv` or equivalent
   - Copy `.envrc.example` to `.envrc` and configure as needed
   - Install pre-commit hooks: `pre-commit install`

## Development Workflow

1. Confirm the application builds with `make build`.
2. Run comprehensive code quality checks with `make code-quality` (includes `go vet` and `golangci-lint`).
3. Execute all tests with `make test`.
4. For integration tests: `make test-integration` (requires Proxmox environment).
5. Keep the working tree clean before finishing.

## Quick Reference

| Task | Command |
|------|---------|
| Build | `make build` |
| Run all checks | `make code-quality && make test` |
| Fast iteration | `make test-quick` |
| Integration tests | `PVETUI_INTEGRATION_TEST=true make test-integration` |
| Install hooks | `pre-commit install` |
| View logs | `tail -f ~/.cache/pvetui/pvetui.log` |
| Clean build | `make clean && make build` |
| Development setup | `make dev-setup` |

## Code Quality Standards

- All code must pass `make code-quality` without errors (includes go vet and golangci-lint).
- Maintain test coverage; add tests for new functionality.
- Use table-driven tests where appropriate.
- Mock external dependencies in unit tests.

## Style Guidelines

- Follow idiomatic Go and clean architecture practices.
- Apply Clean Architecture: handlers → services → repositories → domain models.
- Prefer small, focused interfaces and dependency injection via constructors.
- Use interface-driven development; public functions should accept interfaces, not concrete types.
- Document all exported identifiers with comprehensive GoDoc comments including:
  - Package-level documentation explaining purpose and usage patterns
  - Function documentation with parameter descriptions and examples
  - Type documentation with use cases and thread safety considerations
- Handle errors explicitly; wrap errors with context using `fmt.Errorf("context: %w", err)`.

  **Example:**
  ```go
  // Good
  if err := client.StartVM(vmid); err != nil {
      return fmt.Errorf("failed to start VM %d: %w", vmid, err)
  }

  // Bad - no context
  if err := client.StartVM(vmid); err != nil {
      return err
  }
  ```

- Use context propagation for request-scoped values, deadlines, and cancellations.

## Documentation Requirements

- Update `CHANGELOG.md` under **[Unreleased]** section with user-visible changes.
  - **Important**: Only document changes from the previous release, not intermediate development steps
  - Example: If you change a timeout from 5min → 10min → 3min during development, only document the final change (5min → 3min)
  - Focus on user-visible bug fixes, features, and breaking changes
  - Avoid documenting internal refactorings or temporary fixes made during development
- Add GoDoc examples for complex public APIs.
- Update relevant documentation files when changing behavior.

## Commit Standards

- Write concise commit messages (imperative mood, present tense).
- Use simple, descriptive messages stating what was done.
- Include relevant emojis when appropriate.
- Ask about committing after successfully implementing features.

## Security and Performance

- Validate and sanitize all external inputs (especially VM names, IDs, and user commands).
- **Never log credentials**: Passwords, API tokens, or CSRF tokens should never appear in logs.
- Implement proper error handling and timeouts for external calls (default: 30s).
- Use secure defaults for authentication and configuration.
- **File permissions**: Config files with secrets must be 0o600, cache directories 0o700.
- Profile and benchmark performance-critical code paths.
- Always validate array indices and map keys before access.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devnullvoid/pvetui](https://github.com/devnullvoid/pvetui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
