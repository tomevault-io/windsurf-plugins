---
trigger: always_on
description: The repo is a Rust workspace providing tools and components for attesting confidential guests
---

# AGENTS.md

## Project Overview

The repo is a Rust workspace providing tools and components for attesting confidential guests
(Trusted Execution Environments) and delivering secrets to them. It is part of the
[Confidential Containers](https://github.com/confidential-containers) project.

The main components are:
- **KBS** (Key Broker Service) — remote attestation and secret delivery server (Relying Party)
- **Attestation Service (AS)** — verifies TEE evidence (Verifier)
- **RVPS** (Reference Value Provider Service) — manages reference values for evidence verification
- **KBS Client** / **Trustee CLI** — client tools for testing, configuration and key release

The sum of the above components is called "Trustee", it can be deployed as individual services
that are coupled via RPC or as a single all-in-one binary.

## Security Considerations

Remote attestation and secret delivery are security-sensitive operations. Changes to the
codebase should be made with due consideration of security implications, and new code should
be covered by comprehensive tests that validate security properties. We encourage coding
patterns that express invariant conditions in the type system, e.g. using a type state pattern
or newtypes to prevent misuse of APIs. Always consider the attack surface of new features and
strive to minimize it.

## API Stability

The project is packaged and distributed downstream. Trustee doesn't provide machine-enforcable
schemas for all its payloads and protocols yet, but we strive to avoid breakage for consumers
when iterating. If it's unavoidable, we point it out expclicitly in commit messages, so it
can be tracked and aggregated in release notes.

## Relationship to Guest Components

confidential-containers/guest-components is the counterpart to trustee that is running in a
TEE. It provides the attestation evidence and consumes secrets delivered by Trustee. The two
repositories are developed and released in tandem, but they are decoupled and can be used
independently. In the `kbs-client` crate we reference a revision of guest-components for
which we test and guarantee compatibility in any given revision of Trustee.

## Sample Verifiers

It's unlikely that local development will happen in a TEE that is able to produce genuine
attestation evidence. Therefore there are "sample" and a "sample-device" attester/verifier
pairs that can be used as dummy stubs for testing end-to-end flow in local  development and
testing. Any default policies that is shipped as part of the release should make sure to
reject these dummy evidences and release builds of attestation service should exclude the
sample verifier.

## Conventions

### Commits

Commit messages have to contain a subsystem, indicated by a prefix plus colon, e.g.
"verifier: fix hex encoding". This doesn't have to match strictly, but it should still
indicate a general area of the codebase that is being affected, e.g. "kbs:", "as:", "ci:",
etc.

Commits should compile individually and contain atomic changes.

## PRs

PRs names have to contain a subsystem, indicated by a prefix plus colon, similar to commit
messages. We refrain from overloading the PR description with too much information, especially
excessive use of buzzwords and emojis is discouraged. We want to know _why_ a change is
being made, _how_ it is being made, and if applicable potential negative implications and
alternative options that have been considered.

## Repository Structure

```
├── kbs/                        # Key Broker Service
│   ├── docker/                 # Dockerfiles
│   ├── docs/                   # KBS documentation
│   ├── config/                 # Kubernetes manifests
│   ├── test/                   # E2E tests, test data
│   └── Makefile
├── attestation-service/        # Attestation Service
│   ├── docker/                 # Dockerfiles
│   ├── tests/                  # integration tests, OPA policies, evidence templates
│   └── Makefile
├── rvps/                       # Reference Value Provider Service
│   ├── docker/                 # Dockerfile
│   └── Makefile
├── tools/
│   ├── kbs-client/             # KBS client library & CLI
│   └── trustee-cli/            # Unified Trustee CLI tool
│       └── Makefile
├── deps/                       # Shared library crates
│   ├── verifier/               # TEE verifier implementations
│   ├── policy-engine/          # OPA policy engine (regorus)
│   ├── key-value-storage/      # Storage abstraction (PostgreSQL, local)
│   └── eventlog/               # TCG event log parser
├── integration-tests/          # Workspace-level integration tests
├── protos/                     # Protocol Buffer definitions (attestation, reference)
├── hack/                       # Helper scripts (e2e.sh, release-helper.sh)
├── docker-compose.yml          # Full-stack local deployment
├── .github/workflows/          # CI/CD pipelines
└── .devcontainer/              # Dev container configurations
```

## Build & Development

### Prerequisites

- **Rust toolchain**: version pinned in `rust-toolchain.toml` (currently `1.93.0`)
- **Protobuf compiler** (`protoc`) for gRPC code generation
- **OpenSSL** development libraries
- **tpm2-tss** (for TPM verifier features)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [confidential-containers/trustee](https://github.com/confidential-containers/trustee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
