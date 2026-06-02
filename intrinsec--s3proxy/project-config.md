---
trigger: always_on
description: Rules for AI-assisted development on this repository.
---

# AGENTS.md

Rules for AI-assisted development on this repository.

Project: `s3proxy` — transparent S3 proxy that encrypts PutObject bodies with AES-256-GCM
and decrypts GetObject responses using a KEK derived from configuration. Speaks the S3 HTTP
wire protocol on the client side and forwards signed requests to AWS S3 on the backend.

Tier: **C (shared, critical)**.

## Language

Default agent response: English, even if user writes French.
French response only when user explicitly asks French.

Caveman compression **mandatory** for all conversational responses (default level: `full`).
Code blocks, commit messages, PR descriptions, security warnings, irreversible-action
confirmations stay normal prose (Caveman auto-clarity rules). Do not disable Caveman unless
user says "stop caveman" or "normal mode".

HARD RULE: all code, comments, identifiers, doc strings, commit messages, ADRs, technical
docs in English — every project type, regardless of team spoken language.
User-facing strings + UI copy exempt — match audience language.

## Workflow Skills (mandatory)

Every agent session in this repo must load + apply these skill packs:

- **superpowers** — process discipline (`brainstorming`, `writing-plans`, `executing-plans`,
  `test-driven-development`, `systematic-debugging`, `verification-before-completion`,
  `requesting-code-review`).
- **caveman** — response compression (see Language section).

Pack missing? Install per iagen-dev `INSTALL.md` before work.

### Session start gate

Before any response, clarification, repository inspection, shell command, or file edit:
run `superpowers:using-superpowers` first, then run `caveman` so compression is active
for every response. Use `superpowers:using-superpowers` to decide which additional
skills apply, then follow the selected skill workflows.

### Plan-writing mandatory before non-trivial implementation

Any feature, refactor, bugfix touching more than one function, or agent cannot reason in
one pass:

1. Run `superpowers:brainstorming` — clarify intent + requirements.
2. Run `superpowers:writing-plans` — persist plan at `docs/superpowers/plans/<short-name>.md`
   (commit to git).
3. Execute via `superpowers:executing-plans` (single-session) or
   `superpowers:subagent-driven-development` (parallelisable steps).
4. Gate completion with `superpowers:verification-before-completion` — no "done" claim
   without evidence (test output, lint output, build output).

**Trivial edits exception:** typos, single-line config tweaks, self-evident one-liners
skip steps 1–3 but still verify before claiming done.

### Bug fixes go through systematic-debugging

Any bug, failing test, unexpected behaviour → `superpowers:systematic-debugging` first.
No symptom patching without root cause.

### Code review before merge

Before merge or PR for non-trivial work: run `superpowers:requesting-code-review`.

## Code Quality

After modifying any Go file: run `golangci-lint run ./...` before marking work complete.
Fix all lint errors, re-run until clean. Lint errors = task not done.
`gofmt` non-negotiable — zero diff allowed. Run `gofmt -w .` if in doubt.

## Vulnerability Scanning

After modifying `go.mod` / `go.sum`: run `govulncheck ./...` before marking work complete.
(`vendor/` is honored via `GOFLAGS=-mod=vendor` in env; govulncheck has no `-mod` CLI flag.)
Fix called vulns: `go get <module>@<fixed>`, `go mod tidy`, re-vendor if applicable,
re-run until clean. Imported-only vulns: report to user.
Called vulns remaining = task not done.

## Dependency Management

Any `go.mod` change → run `go mod tidy` then `go mod vendor`.
`vendor/` must be committed — never gitignored.
CI uses `go build -mod=vendor`. Never `go get` inside Docker build without re-vendoring after.

**`vendor/` is read-only.** Never edit files under `vendor/` by hand — not to patch a bug,
not to silence a lint warning, not to "just try something". Upstream-only fixes:
`go get <module>@<fixed-version>` + `go mod tidy` + `go mod vendor`. If upstream lacks
a needed fix, fork the module, point `replace` at the fork in `go.mod`, then re-vendor.
Hand-edits to `vendor/` get blown away on the next `go mod vendor` and silently mask
supply-chain provenance.

## Generated Code

Generated sources are **read-only**. Never hand-edit files produced by a code generator:

- `protoc` / `buf` outputs for gRPC + Protobuf (typically `*.pb.go`, `*_grpc.pb.go`,
  often under `gen/`, `pb/`, or `proto/`)
- `mockgen` / `moq` mocks
- `sqlc`, `ent`, `gqlgen`, `wire_gen.go`, `oapi-codegen`, `swag` outputs
- any file with a `// Code generated ... DO NOT EDIT.` header

To change generated code: change the source of truth (`.proto`, `.sql`, schema, interface)
then re-run the generator (`buf generate`, `go generate ./...`, `sqlc generate`, etc.).
Commit the regenerated files alongside the source change in the same commit.

Generated files are committed (not gitignored) so builds and reviews are reproducible.
CI must regenerate and `git diff --exit-code` to catch drift between source + output.

## Testing & Architecture

Red-Green-Refactor: failing test first, then implementation.
DI via constructors — no package-level globals, no `init()` side effects.
Small, focused interfaces at call site. Never inject concrete type where interface suffices.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Intrinsec/s3proxy](https://github.com/Intrinsec/s3proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
