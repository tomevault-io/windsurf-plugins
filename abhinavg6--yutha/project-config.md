---
trigger: always_on
description: This file is read by AI coding agents (Cursor, Claude Code, Codex, etc.) working inside this repository. It orients an agent to the codebase, lists the build and test commands, and calls out conventions that the rest of the repo assumes you already know.
---

# Notes for AI coding agents

This file is read by AI coding agents (Cursor, Claude Code, Codex, etc.) working inside this repository. It orients an agent to the codebase, lists the build and test commands, and calls out conventions that the rest of the repo assumes you already know.

If you're a *human* contributor looking for the contribution guide, that lives at [`docs/community/CONTRIBUTING.md`](docs/community/CONTRIBUTING.md). If you're an *AI agent reading the published docs at yutha.ai* (not the code), look at `/llms.txt` and `/llms-full.txt` on the site — they're written for you.

## What Yutha is

Open-source infrastructure for groups of AI agents. Identity, capability, accountability, declarative norms (Cedar+), optional cryptographic verifiability (Sui anchoring). Framework-agnostic — runs in front of agents built in LangGraph, CrewAI, or anything else. The full conceptual overview is at `docs/index.md`.

## Repository layout

```
/spec        — wire & artifact specs (RFC-governed; the contract)
/crates      — Rust workspace: control plane, registry, capability, transport,
               receipts, Cedar+ engine, Signer trait + external Signer backends
               (Vault, GCP KMS, Azure Managed HSM), Attestor trait + external
               Attestor backends (SPIFFE, OIDC), preview tooling (yutha-replay,
               yutha-sim, yutha-diff), ops CLI, proto crate, conformance suite
/backends    — Pluggable backends: postgres-receipt (production receipt
               store), sui-anchor (optional verifiability layer)
/contracts   — Move package for Sui receipt anchoring (sources/, tests/)
/sdks        — Framework adapters (sdks/python/ ships today: LangGraph,
               CrewAI, OpenAI Agents, Microsoft Agent Framework). Latest
               release: yutha 0.1.0a4 on PyPI.
/interop     — Cross-language differential testing (interop/go/)
/docs        — Source for the MkDocs Material site published at yutha.ai.
               docs/internal/ holds engineering reference docs that aren't on
               the published nav (PRD, threat model, build plan, constitution
               design memo, conformance suite, ADRs, per-release notes). The
               canonical release record lives on GitHub Releases.
/scripts     — Repo tooling (e.g. build-llms-full.py)
```

The wire-level contract lives in `/spec/`. The Rust implementation in `/crates/` is the reference, not the spec. When implementation and spec disagree, the spec wins and the implementation is the bug.

## How to build and test

The workspace pins to a local clone of [MystenLabs/sui-rust-sdk](https://github.com/MystenLabs/sui-rust-sdk) at a specific commit (see the comment block in `Cargo.toml`). The clone must sit as a sibling to this repo at `../sui-rust-sdk`. CI clones it automatically; locally you need:

```bash
cd ..
git clone https://github.com/MystenLabs/sui-rust-sdk
git -C sui-rust-sdk checkout $(grep SUI_RUST_SDK_REV yutha/.github/workflows/ci.yml | awk '{print $2}')
cd yutha
```

Then:

```bash
# Rust (workspace).
cargo build --workspace --all-targets
cargo test --workspace --all-features
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo fmt --check
cargo deny check

# Conformance suite (in-memory backend).
cargo test -p yutha-conformance

# Python SDK + adapters.
cd sdks/python
uv sync --extra dev --extra crewai --extra openai-agents --extra maf
uv run pytest                          # unit tests
uv run pytest -m integration -v        # integration (needs control plane up)
uv run ruff check .
uv run mypy src/yutha
cd ../..

# Move package (Sui contracts).
cd contracts/sui/receipt_anchor
sui move test --build-env testnet
cd ../../..

# Docs site.
pip install -r requirements-docs.txt
python3 scripts/build-llms-full.py     # regenerates docs/llms-full.txt
mkdocs serve                            # preview at http://127.0.0.1:8000

# Cross-language interop (Go differential).
cd interop/go && make all
```

For the integration tests, bring the control plane up in a separate terminal first:

```bash
export YUTHA_BOOTSTRAP_SEED=$(openssl rand -hex 32)
export YUTHA_OPERATOR_PUBLIC_KEY=$(cargo run -p yutha-ops --quiet -- print-operator-pubkey)
cargo run -p yutha-control-plane -- \
  --admission-mode open \
  --workload support-queue \
  --operator-public-key "$YUTHA_OPERATOR_PUBLIC_KEY"
```

## Conventions to honor

**Spec-first.** Changes to the wire format, artifact format, or any externally-observable behavior require an RFC. See `docs/community/RFC_PROCESS.md`. Don't ship a wire-format change as an implementation PR; reviewers will bounce it.

**No backcompat ceremony pre-public-launch.** Until the repo flips public for Phase 2, prefer breaking changes over opt-in/fallback paths. The codebase has no shipped consumers yet; carrying compat shims is purely cost.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abhinavg6/yutha](https://github.com/abhinavg6/yutha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
