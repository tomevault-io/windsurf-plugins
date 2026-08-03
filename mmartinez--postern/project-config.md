---
trigger: always_on
description: A credential-brokering HTTPS forward proxy for AI agents. The agent calls outbound APIs without auth headers (or with placeholder values); postern matches the request host against YAML-declared broker rules, resolves the rule's credential reference from a pluggable credential provider (1Password Service Accounts, Bitwarden Secrets Manager), and injects it before forwarding. The agent never holds the real credential.
---

## What postern is

A credential-brokering HTTPS forward proxy for AI agents. The agent calls outbound APIs without auth headers (or with placeholder values); postern matches the request host against YAML-declared broker rules, resolves the rule's credential reference from a pluggable credential provider (1Password Service Accounts, Bitwarden Secrets Manager), and injects it before forwarding. The agent never holds the real credential.

Apache-2.0 licensed.

## Development environment — container-first, non-negotiable

The host needs only Docker, git, and the devcontainer CLI. **No Go toolchain on the host.** Every Go, lint, test, format, and vuln tool lives inside the devcontainer, version-pinned by `.mise.toml`.

- Start the env once: `devcontainer up --workspace-folder .`
- Drop into a shell: `make shell`
- Anything Go-related goes through `make <target>`. From the host these wrap `devcontainer exec`; inside the container they are direct invocations. The Makefile auto-detects which side it's on.
- Bump tool versions in `.mise.toml` only. CI reads the same pins.

If a command fails on the host with "command not found", the fix is to run it via `make` (or inside `make shell`), not to install the tool globally.

## Commands

All Go work runs through `make`; on the host these wrap `devcontainer exec`.

- `make shell` — interactive shell inside the devcontainer.
- `make build` — build `dist/postern`.
- `make test` — full suite with `-race` and coverage.
- `make lint` — golangci-lint.
- `make ci` — lint + test + vuln + license check (what CI runs); run before pushing.
- `make snapshot` — local release build (binaries, archives, SBOMs, checksums; no publish/sign).
- `make licenses` — regenerate `THIRD_PARTY_NOTICES.md` after any dependency change.

## Project layout

- `cmd/postern/` — CLI entry point.
- `internal/broker/` — rule engine, the goproxy hook, credential injection, hot-reload.
- `internal/proxy/` — the MITM proxy (goproxy) and request handler.
- `internal/ca/` — local CA: generate, mint per-host leaf certs, OS trust-store integration.
- `internal/config/` — YAML schema, strict loader, line-numbered validation.
- `internal/credstore/` — provider registry plus the `onepassword/` and `bitwarden/` backends.
- `internal/token/` — service-account token storage (file and OS keyring).
- `internal/{runtime,logging,templates,version}/` — server assembly, slog setup, template rendering, build version.

## Git workflow

- **Commits run inside the devcontainer.** Lefthook hooks invoke gofumpt, golangci-lint, gitleaks, and a banned-strings check; the host has none of those.
- Commit messages follow **Conventional Commits**: `feat`, `fix`, `chore`, `docs`, `test`, `refactor`, `build`, `ci`, `perf`, optionally with a scope (`feat(config): ...`). Enforced by the commit-msg hook.
- **Never bypass hooks.** No `--no-verify`. If a hook fails or hangs, fix the underlying issue — the hooks exist to catch bugs and leaks early.

## Working on a task

The project is sliced **vertically**: each task delivers a user-observable capability end-to-end through whatever layers it needs. No horizontal "build all models first, all handlers next" phases.

Per slice:

1. State scope before opening files. For non-trivial work, surface 2–3 viable approaches with tradeoffs and wait for review before coding.
2. **TDD.** Failing test first → confirm RED → minimum implementation to GREEN → refactor with tests still green. Skip TDD only for pure docs/config changes with no behavioral impact.
3. **Verify library/SDK shape against current docs before coding against it.** Training data is stale for the 1Password SDK, goproxy streaming semantics, 99designs/keyring backend enumeration, and several others.
4. **Coverage gate:** ≥ 80% on the core packages (broker, config, token, onepassword). Lower bar acceptable on glue code that is already exercised by integration tests.
5. One slice, one commit. Conventional commit message.

## Go conventions

We follow **Effective Go** and the **Google Go Style Guide**. The authoritative enforcer is `golangci-lint` (see the enabled linter set in `.golangci.yml`). The points below are the project-specific tightenings that don't fall out of lint automatically.

### Naming

- MixedCaps, never `snake_case` or `SHOUTING_CASE` for Go identifiers. (YAML values and env var names are strings, not identifiers, and follow their own conventions.)
- Initialisms stay uppercase as a block: `CacheTTL`, `URL`, `ID`.
- No `Get`-prefixed getters. The field `X` is read as `.X` or via a method `X()`, never `GetX()`.
- Constructors are `New<Type>` for the package's primary type, `New<Type><Variant>` for alternatives.
- File names are nouns (`schema.go`, `validator.go`, `cache.go`), not verbs.
- Receiver names are 1–2 characters and **consistent across all methods of a type**.
- Package names are short, lowercase, no underscores, no plurals.

### Comments


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmartinez/postern](https://github.com/mmartinez/postern) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
