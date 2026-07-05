---
trigger: always_on
description: Guidance for any Claude session working in this repository.
---

# CLAUDE.md

Guidance for any Claude session working in this repository.

## Project

`ans` is an open-source implementation of the Agent Name Service:
a Registration Authority (`ans-ra`), an append-only Merkle-tree
Transparency Log (`ans-tl`), an offline verifier (`ans-verify`),
and a development DNS server (`ans-dns`). Written in Go.

The wire shape (event envelope, SCITT COSE receipts, sumdb-note
checkpoints, `/root-keys` format) is the public contract; offline
verifiers and TL clients depend on it byte-for-byte.

## Non-negotiable quality bar

Everything that lands on `main` must meet production-grade standards:

- **Best practices only.** Idiomatic Go, SOLID, hexagonal
  domain/port/adapter boundaries. No global mutable state, no panics
  in request paths, no swallowed errors, no `fmt.Println`/`log.Printf`
  in library code.
- **Sufficient logging is required.** Every package ships with enough
  structured logging (`zerolog`) to debug it in production from the
  logs alone — a silent component is a non-negotiable defect, not a
  style preference. Concretely:
  - Library/adapter/service code takes an injected logger (a
    `WithLogger(zerolog.Logger)` option or constructor parameter,
    defaulting to `zerolog.Nop()`), tagged with a `component` field. It
    never reaches for the package-global `log` — that's reserved for
    `cmd/*` wiring.
  - Log every meaningful state transition and outcome at `INFO`
    (e.g. order opened, order finalized, agent activated, event
    appended), every recoverable/expected detour at `DEBUG` (retries,
    pending re-drives, cache misses, fallbacks), and every failure —
    especially upstream/provider/store errors — at `ERROR` (or `WARN`
    when the caller will recover), always with `.Err(err)` and the
    identifiers needed to trace the request (`agentId`, `orderRef`,
    `fqdn`, `serialNumber`, leaf index, …). A bare `return err` with no
    log on a path that crosses a trust or network boundary is a defect.
  - Never log secrets, private keys, full CSR/cert bytes, bearer
    tokens, challenge key authorizations, or other sensitive material —
    log stable identifiers and fingerprints instead.
- **Unit test coverage is required.** Every new package ships with
  tests. `internal/domain` stays at 100% of statements.
  `internal/crypto` targets 100% but may sit at ≥95% when the
  remaining uncovered lines are defensive dead-code branches behind
  preceding guards (e.g., `toJWSWireFormat` type-assertion failures
  that `checkAlgMatchesKey` already rejects upstream). Any such
  exception must be annotated in-code with a SAFETY or NOTE comment
  explaining why the branch is unreachable. Overall coverage must
  not drop below 90% as enforced by `make test-cover`, computed
  across `internal/` only — the `cmd/*` entry-point binaries
  (main() + flag wiring + dependency init) are excluded from the
  coverage denominator; their correctness is exercised by
  integration tests and the `scripts/demo/` end-to-end scripts,
  not unit tests.
- **Zero TODOs, hacks, assumptions, or deferred features on `main`.**
  If something is not complete, it does not ship.
- **No placeholder routes.** A route registered on the HTTP router on
  `main` is a committed contract. If the real implementation depends
  on a piece of work that isn't ready, the route is NOT registered.
  A placeholder response on a live route creates a DTO that downstream
  clients will encode against, and replacing it later is a breaking
  change. The correct state is: unimplemented routes are unregistered,
  and the 404 is the signal.
- **Pre-implementation shape diff.** Before writing a handler, DTO,
  or wire-format envelope, locate the corresponding struct/schema in
  the canonical source — the OpenAPI contracts under `spec/` — and
  paste it with a citation into the change description. Every field
  that differs must be called out explicitly. This is how we catch
  shape drift before it reaches the wire, where it would become a
  breaking change for verifiers and clients.
- **`make check` must pass** (`fmt`, `vet`, `golangci-lint`,
  `test-cover`). CI blocks on any failure.
- **No AI `Co-Authored-By:` trailers.** Do not append
  `Co-Authored-By: Claude …`, `Co-Authored-By: Copilot …`, or any
  similar trailer naming an AI assistant on commits in this repo.
  This project follows the Developer Certificate of Origin: every
  author listed on a commit is asserting the DCO, which only a real
  person can do, and noreply@*.com addresses cannot sign off. The
  human running the tooling is the sole author. Use `Signed-off-by:`
  trailers (added automatically by `git commit -s`) to satisfy DCO;
  every commit on `main` must be both DCO-signed-off and GPG-signed.

## V1 vs V2 RA APIs

`ans` serves **both** the V1 and V2 RA APIs side by side. Two
independent lanes from the handler all the way down to the TL:

- **V1 lane**: `POST /v1/agents/register`, `GET /v1/agents/{agentId}`,
  `POST /v1/agents/{agentId}/verify-acme`, `POST
  /v1/agents/{agentId}/verify-dns`, `POST
  /v1/agents/{agentId}/revoke`, the five
  `/v1/agents/{agentId}/certificates/…` cert-management routes, and
  the four `/v1/agents/{agentId}/certificates/server/renewal{,
  /verify-acme}` renewal routes. V1 handlers stamp `SchemaVersion =

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentnameservice/ans](https://github.com/agentnameservice/ans) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
