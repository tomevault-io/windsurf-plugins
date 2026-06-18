---
trigger: always_on
description: `tap-sdk` is the Go source-of-truth implementation of the Taproot Assets
---

# AGENTS.md — tap-sdk

## Project Overview

`tap-sdk` is the Go source-of-truth implementation of the Taproot Assets
application SDK. It exposes typed wallet, issuer, universe, proof, burn, and
transfer surfaces over `tapd` without requiring application developers to deal
with low-level protobuf types.

**Repository:** `github.com/lightninglabs/tap-sdk`
**Language:** Go (source of truth for future language SDKs)
**Status:** Pre-v1.0 (breaking API changes possible)

## Architecture

```
tap-sdk/
├── Root package (tapsdk)
│   ├── wallet.go              — High-level Wallet entrypoint
│   ├── tx_builder.go          — Address-based transfer builder
│   ├── interactive_tx_builder.go — Interactive (keyless) transfer builder
│   ├── asset.go              — Public asset, collection, and issuance types
│   ├── clients.go             — Client interface definitions
│   └── errors.go              — SDK error types and sentinels
├── grpc/                      — gRPC client implementation
├── rest/                      — REST client implementation
├── internal/vpsbt/            — Virtual PSBT encoding
├── internal/codec/            — Cryptographic utilities (alt-leaves, STXO)
├── macaroon/                  — Authentication helpers
└── docs/                      — User and contributor documentation
    └── design/                — Durable design decisions
```

### Key Design Principles

1. **SDK types are the only public surface.** Users never import `taprpc`
   or other `taproot-assets` packages.
2. **Transports are implementation details.** All proto conversions happen at
   the `grpc/` and `rest/` package boundaries.
3. **Wallet is the entrypoint.** High-level operations go through `Wallet`,
   low-level through the `Client` interface.
4. **Public types live at the root.** Fixed-size byte arrays and business
   structs are discoverable from the main `tapsdk` import.
5. **The SDK is opinionated.** The public surface should not blindly mirror
   raw `taprpc` semantics when a cleaner Taproot Assets model exists.
6. **Design docs are durable.** `docs/design/` captures accepted API and
   architecture decisions, not short-lived implementation plans.

### Asset Identity Model

The SDK must distinguish between the protocol's low-level identifiers and the
user-facing identity model:

- **Fungible assets are identified by group key** in the normal SDK surface.
  The `asset_id` is tranche-level and should not be the primary identifier
  that application developers work with for fungible assets.
- **Collectibles / non-fungible assets are identified by asset ID.** In that
  case the tranche is the asset, so exposing the asset ID is correct.
- Low-level client wrappers may still need to translate raw RPC fields, but
  high-level APIs and docs should consistently present the semantic model:
  fungible => group key, non-fungible => asset ID.

### Address Defaults

The SDK should expose an opinionated default address flow:

- **V2 addresses are the default** for the normal, high-level receive/send
  surface unless there is a concrete reason to do otherwise.
- Older address versions are considered advanced or compatibility paths and
  should remain available through lower-level client methods instead of being
  promoted in the primary UX.

## Development

### Build Commands

| Command | Purpose |
|---------|---------|
| `make build` | Build the SDK |
| `make lint` | Run golangci-lint (via Docker) |
| `make fmt` | Format code (gosimports + gofmt) |
| `make unit` | Run all unit tests |
| `make unit pkg=<pkg>` | Run tests for specific package |
| `make unit pkg=<pkg> case=<test>` | Run a specific test case |
| `make itest-main` | Integration tests against tapd built from `taproot-assets` main (local-dev default) |
| `make itest` | Integration tests against the pinned released tapd image (what CI uses) |
| `make itest-run-main case=<test>` | Run a single itest against tapd-main without re-spinning the stack |

### Integration tests

Prefer `make itest-main` locally. It builds tapd from the latest
`taproot-assets` main so proto/RPC changes in tapd are exercised before
CI catches them. CI runs `make itest` (pinned released image) because
bleeding-edge tapd may not have a pre-built image yet — only reach for
plain `itest` when reproducing CI.

### Code Style

Follow Lightning Labs conventions (see `.gemini/styleguide.md`):

- **80 character line limit** (tabs count as 8 spaces)
- **Every exported function** must have a comment starting with function name
- **Comments explain "why"**, not "what"
- **Table-driven tests** with `require` assertions
- **Logical stanzas** separated by blank lines
- **No AI watermarks** — no `Co-authored-by: Claude` or similar

### Commit Style

Format: `subsystem: short description`

Keep commit messages concise. **Commits are not PR descriptions.** The
subject line should be under ~70 characters; if a body is needed, aim
for 2–4 short sentences — not paragraphs, not bulleted lists, not
design discussions. Explain the *why* when it isn't obvious from the
diff; skip the body entirely when the subject already says it.

PR descriptions belong in the PR body (with context, test plan, etc.),
not in every commit. A reader running `git log --oneline` should be

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lightninglabs/tap-sdk](https://github.com/lightninglabs/tap-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
