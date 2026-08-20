---
trigger: always_on
description: Guidance for coding agents working in this repository. Keep it accurate as the
---

# AGENTS.md

Guidance for coding agents working in this repository. Keep it accurate as the
project evolves. For human-facing usage and threat model, see `README.md`.

## Project overview

`git-remote-cloak` is a git remote helper (Go) that stores an end-to-end
encrypted mirror of a repository on any ordinary git host. The host sees only
opaque AEAD-encrypted blobs (age v1 format, ChaCha20-Poly1305, single shared
symmetric key); local repositories stay plain git. It is a ground-up
replacement for `git-remote-gcrypt` with safe concurrent pushes
(compare-and-swap), rollback/replay detection, repo-identity pinning, and
typed error reporting.

- Module: `github.com/b4ryon/git-remote-cloak`, Go 1.26+.
- Entry point: `cmd/git-remote-cloak` (the helper binary; also symlinked as
  `git-cloak` for the CLI subcommands).
- The helper speaks git's remote-helper protocol over stdin/stdout for
  `cloak::` URLs; the `git-cloak` symlink exposes user commands (`version`,
  `status`, `keygen`, `repack`, `rekey`, `key export|import|delete`,
  `accept-rollback`, `accept-repo-change`, and the debug
  `encrypt|decrypt|seed-remote` subcommands).

## Setup and build commands

Prerequisites: Go 1.26+ and git. On macOS, Xcode command line tools
(`xcode-select --install`) are needed for the cgo Keychain/Touch ID key
backend; Linux builds are pure Go and use a key file.

- Build: `make build` (outputs `bin/git-remote-cloak` and the `bin/git-cloak`
  symlink).
- Install into `~/bin`: `make install` (override `PREFIX=`).
- Format: `make fmt` (runs `gofmt -l -w` over `cmd internal test`).
- Vet: `make vet`.
- Vulnerability scan: `make vuln` (needs `govulncheck` installed and network
  access to vuln.go.dev).
- Clean: `make clean`.

Do not run `go build`/`go install` with a hand-written version. The build
version is injected from the latest git tag via `-ldflags`; see Versioning.

## Testing instructions

Always run tests through the `make` targets, not a bare `go test ./...`. The
targets export `TMPDIR=$HOME/tmp`, and the integration `TestMain` hard-refuses
to run unless `TMPDIR` is under `~/tmp`. A plain `go test ./test/...` fails with
a "temp dir is ..." error by design.

- Unit suite: `make test` (runs `./internal/...` with `-count=1`).
- Integration + security suites: `make test-integration` (rebuilds the helper
  first; `-count=1` is required because the test cache does not track the
  rebuilt binary).
- Race detector over in-process suites: `make test-race`.
- macOS Keychain backend: `make test-darwin` (build tag `darwinkeystore`).
- End-to-end suite: `make test-e2e` (build tag `e2e`).
- Full gate before declaring done: `make check` (vet, lint, vuln, test,
  test-integration, test-darwin).

When fixing a bug, add a regression test that fails against the old code and
passes against the fix. Prefer reproducing failures end-to-end (integration or
security suite) where the change touches fetch/push/protocol behavior.

## Code style

- Standard Go: format with `gofmt`; `make fmt` and `make vet` must be clean.
- Match surrounding code: naming, comment density, and idioms. Read the file
  before editing.
- Every generated source file starts with a short comment block describing what
  the file does.
- Error handling (load-bearing in this codebase, treat it as a contract):
  - Wrap leaf errors with operation context using `fmt.Errorf("...: %w", err)`
    so the cause chain is preserved (`%w`, never `%v`).
  - Classify domain errors with `cloakerr.New`/`cloakerr.Newf` and the right
    `Kind` (`Auth`, `Network`, `RepoNotFound`, `Tamper`, `Rollback`,
    `CASExhausted`, `LocalGit`, `KeyUnavailable`, `Crypto`, `Protocol`).
  - Detect sentinels through `errors.As`/`errors.Is`, not direct type
    assertions like `err.(*gitx.GitError)`, so a wrapped error is still
    recognized (see `config.isNoMatchingKeys`, `backend.isMissingPacksTree`).
  - Never silently swallow an error. Do not discard error returns with `_`, and
    when reading streams check `bufio.Scanner.Err()` after the loop (a `false`
    from `Scan()` can be a real read failure, not a clean EOF).
  - Fail closed: on any ambiguity about integrity, identity, or rollback state,
    refuse rather than proceed.

## Project structure

- `cmd/git-remote-cloak` - binary entry point and protocol dispatch.
- `internal/cli` - `git-cloak` subcommands and argument handling.
- `internal/helper` - git remote-helper protocol loop (fetch/push batches).
- `internal/engine` - fetch/decrypt/index and push/encrypt/consolidate logic.
- `internal/backend` - reads/writes encrypted state on the host via git.
- `internal/agecrypt` - age encryption, the pack writer, decrypt.
- `internal/keystore` - key load/save/delete (file backend, macOS Keychain).
- `internal/manifest` - the AEAD manifest (pack list, generation, repo id).
- `internal/state` - local per-remote bookkeeping (rollback pin, repo-id pin,
  applied-pack set).
- `internal/config` - `cloak.*` git config parsing.
- `internal/setup` - session open/validation (CheckPin, CheckRepoID).
- `internal/cloakerr` - typed error Kinds and helpers (`KindOf`, `WithHintOn`).
- `internal/gitx` - thin wrapper around invoking git.
- `internal/geometry`, `internal/logx`, `internal/userpresence`,
  `internal/version` - support packages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b4ryon/git-remote-cloak](https://github.com/b4ryon/git-remote-cloak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
