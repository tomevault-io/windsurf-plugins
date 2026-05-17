---
trigger: always_on
description: Teep is a secure LLM inference API proxy (Go, stdlib `testing`, no frameworks):
---

# Code Agent Instructions for teep

Teep is a secure LLM inference API proxy (Go, stdlib `testing`, no frameworks):

- Teep verifies that API endpoints are running expected docker images in a CVM.
- Teep ensures requests and responses are encrypted at all times.
- Teep ensures this encryption is fully authenticated by TEE attestation.

Teep is designed to BLOCK REQUEST ACTIVITY when enforced validation factors fail.

## Data Flow

The Teep proxy receives an OpenAI-compatible chat request → resolves model to provider →
fetches and validates TEE attestation per policy → forwards (or blocks) the request.

The proxy receives concurrent API inference requests to multiple models from multiple client API consumers simultaneously, and should support expansion to handle multiple concurrent providers. All code paths from the HTTP handler inward must be safe for concurrent use. All attestation caches, key pinning, connection pinning, supply chain validation, and supply chain caches must also be safe for concurrent use via multiple clients performing simultaneous access of multiple providers and models.

## Key Code Directories

- `cmd/teep/` — CLI entry point, subcommands (`serve`, `verify`), flag definitions.
- `internal/proxy/` — HTTP handler that accepts OpenAI-compatible requests and routes to providers.
- `internal/provider/` — Per-provider attestation and connection logic (subdirs: `nearcloud/`, `neardirect/`, `chutes/`, `venice/`, `nanogpt/`, `phalacloud/`).
- `internal/attestation/` — TDX, NVIDIA, sigstore, Rekor, and supply-chain verification.
- `internal/e2ee/` — End-to-end encryption sessions and relay logic.
- `internal/config/` — Configuration parsing and strict validation.
- `internal/verify/` — Orchestrates multi-factor verification and report generation.
- `internal/multi/` — Concurrent multi-provider verification.

## Core Commands

- Run local tests: `make check` (quick: fmt + vet + lint + unit tests).
- Run full integration tests: `make integration` (slow; optional API keys or config).
- Generate provider verification reports: `make reports` (requires API keys or config).

## Git Workflow

This repository is managed by Git and hosted on GitHub.

- For multi-phase plans, use one commit per phase.
- Ensure new code has unit test coverage before committing.
  - Run `make check` before each commit.
  - Stage only specific files you modified. Do not use `git add .` or `git add -A`.
- Ensure major features have integration test coverage upon plan completion.
  - Run `make integration` and `make reports` when finishing a plan or any major change.
- Do not mention audit identifiers in code or commit messages.

## Repository Rules

Teep is *critical infrastructure security software* for handling *highly confidential data*.

**The measure of this software's correctness is how strictly it evaluates and authenticates providers, not how many providers pass factor authentication checks or provide service.**

To ensure data confidentiality and integrity, adhere to the following rules:

### Always Fail-Closed

Failing closed is a FEATURE, not a BUG. It is more important to protect confidential traffic than it is to provide service. Provider verification failures are not bugs.

- Validation issues of any kind must FAIL LOUDLY AND FAIL CLOSED.
- Failed validation MUST block requests unless specifically whitelisted by `allow_fail`, by `--force` (debug builds only: bypasses all enforced factors), or by `--offline` (skips network-dependent checks such as Intel PCS, NRAS, sigstore, and Proof of Cloud).
- Error paths MUST only return or propagate errors. Any other behavior is a defect.
- Reject malformed input entirely; never silently drop malformed elements.
- Unknown, misspelled, ambiguous, or semantically invalid config values MUST be rejected at startup.
- JSON unmarshalling MUST use the internal/jsonstrict parser.
- All low-level parsers MUST return unknown field names to callers instead of logging or deduplicating them internally. Callers own the policy decision to fail, warn once per logical operation, or use lower-severity logging in hot paths.
- If you can't make development progress due to a failing validation, STOP and ask for advice.
- Fail loudly, not silently: when an expected verification step is skipped because prerequisites are missing, malformed, or unexpectedly unavailable, emit a clear non-secret diagnostic at warn level or stronger.

### Always Ensure Attestation Integrity

- Attestation MUST be verified before any request is forwarded.
- Use `Connection: close` to prevent TLS connection reuse across attestation boundaries.
- Nonces MUST originate from the client, not the server response.
- Never trust provider-asserted "verified" fields without independent cryptographic verification.
- Cache misses MUST trigger full re-attestation, never pass-through.
- Cache eviction MUST NOT allow unattested connections.
- Provider and model routing MUST ensure uniqueness and determinism.

### Always Ensure Cryptographic Safety

- All cryptographic comparisons MUST be constant-time (`subtle.ConstantTimeCompare`). Never use `==`, `!=`, `bytes.Equal`, or `strings.EqualFold` on secrets, keys, fingerprints, nonces, or hashes.
- ALWAYS authenticate encryption keys via attestation binding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [13rac1/teep](https://github.com/13rac1/teep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
