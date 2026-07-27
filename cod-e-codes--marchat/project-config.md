---
trigger: always_on
description: Rules for the marchat terminal chat project
---


# AI Assistant Rules for marchat

## Core Principles
- You are a collaborative coding assistant for marchat, a Go-based terminal chat application
- Prioritize code quality, security, and maintainability over speed
- When uncertain about marchat's architecture, read `ARCHITECTURE.md` / `PROTOCOL.md` or ask for clarification
- Use ASCII hyphen `-` in rules and project markdown, not Unicode em dash

## Agent workflow (required)
Follow `.cursor/skills/README.md` pipeline on every substantive task:

1. **Scope** - read `developing-marchat` plus the domain skill (`client-marchat`, `server-marchat`, etc.)
2. **Implement** - extend existing code; verify APIs from `go.mod`, repo source, or official docs (web search when needed), not training-data defaults
3. **Validate** - `gofmt`, `go vet`, `go test ./...`, `go test -race ./...`, nested `plugin/sdk` per `testing-marchat`
4. **Document** - `CHANGELOG.md` / `TESTING.md` when behavior or coverage shifts (`writing-marchat-docs`)
5. **Close** - read `git-workflow-marchat` and **always** provide a conventional commit message covering **all** uncommitted files (`git status` / `git diff`), even when the user did not ask to commit

**Never** weaken tests, skip assertions, or add conditional passes to green CI. **Do not** `git commit` or `git push` unless the user explicitly asks.

## Agent skills
Project skills live in `.cursor/skills/` (index: `.cursor/skills/README.md`). Cursor discovers them automatically; invoke with `/skill-name`. **Read and follow the matching skill before specialized work.**

| Skill | Use when |
|-------|----------|
| `developing-marchat` | Implementing or refactoring Go code, toolchain validation |
| `testing-marchat` | Tests, `-race`, coverage, CI DB smoke, `plugin/sdk` module |
| `debugging-marchat` | `-doctor`, env/DSN, connection or E2E issues |
| `releasing-marchat` | Version bumps, GitHub release, packaging, Docker |
| `writing-marchat-docs` | CHANGELOG, README, ARCHITECTURE, PROTOCOL, TESTING |
| `git-workflow-marchat` | Commit message and PR drafts at task end (no commit/push unless asked) |
| `database-marchat` | `db.go`, dialect SQL, SQLite/Postgres/MySQL schema |
| `protocol-marchat` | Wire types, E2E encoding, `PROTOCOL.md` |
| `client-marchat` | `client/` TUI, commands, keystore, websocket |
| `server-marchat` | `server/`, `cmd/server/`, hub, admin web |
| `plugins-marchat` | `plugin/` SDK, host, manager, licenses |

Prefer this repo (`go.mod`, `ARCHITECTURE.md`, `PROTOCOL.md`, skills) as source of truth. For Cursor Agent Skills layout, GitHub Actions, or dependency APIs not covered here, web search or fetch official docs before changing config.

## Project Context
- Language: Go 1.25+ with modern idioms (see `go.mod` for toolchain patch; CI/Docker/docs stay aligned)
- Architecture: Client-server with WebSocket (JSON) communication
- Key components: Client TUI (Bubble Tea / Lipgloss), server hub + WebSocket handlers, shared types, plugin subprocess host, optional web admin (TUI + HTTP), health endpoints
- Database: Pluggable SQL at runtime via `MARCHAT_DB_PATH` - SQLite (default), PostgreSQL, or MySQL; dialect-aware schema and queries in `server/db_dialect.go` and `server/db.go`. Durable tables include reactions and read receipts (not only message rows). Use `mysql:` / `mysql://` DSN forms so driver detection matches Postgres-style URLs (see CI smoke tests and docs).
- Security (chat E2E): **Global shared symmetric key** (32 bytes), **ChaCha20-Poly1305** for message/file payloads on the wire; server stores and relays opaque ciphertext (see `PROTOCOL.md`, `shared/crypto.go`). **Not** per-user X25519 key exchange for chat - key distribution is out-of-band (`MARCHAT_GLOBAL_E2E_KEY`, or copy `keystore.dat` + passphrase)
- Keystore (client): Passphrase → **PBKDF2** (SHA-256, 100k iterations) → **AES-GCM** wraps JSON holding `global_key`. **v3** files: magic `marchatk` + format byte + **random salt** in header (portable; legacy path-as-salt files **migrate** on load). See `client/crypto/keystore.go`
- Plugin licenses: Ed25519 signing/validation (`plugin/license/`, `cmd/license`) - separate from chat E2E
- Deploy / TLS: Client supports **WSS behind reverse proxies** (e.g. Caddy); see `deploy/`, `docker-compose.proxy.yml`, README proxy section
- CI/CD: GitHub Actions - `go.yml` (tests, **race** where applicable, **Postgres/MySQL schema smoke** alongside SQLite), `release.yml` (releases; assets via **`gh` CLI**; optional **`publish-downstream-packages`** with `PACKAGING_GITHUB_PAT` / `AUR_SSH_PRIVATE_KEY`; **`CGO_ENABLED=0`** on cross-builds where documented)
- Docker: Multi-arch images (linux/amd64, linux/arm64) published to Docker Hub

## Code Standards
- Follow Go conventions: gofmt, go vet, golangci-lint
- Use Go modules: NEVER manually edit go.mod
- Fix ALL compiler and linter warnings - they are NOT harmless
- Keep functions focused and well-documented
- Use struct embedding and interfaces for extensibility

## Dependencies and Tooling
- NEVER hardcode versions in go.mod
- Always use: go get -u package or go get package@latest
- For new modules: go mod init github.com/username/project

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cod-e-Codes/marchat](https://github.com/Cod-e-Codes/marchat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
