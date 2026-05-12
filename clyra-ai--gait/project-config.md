---
trigger: always_on
description: This file gives coding assistants and contributors the project-wide rules for building **Gait**.
---

# Gait: Agent Instructions (repo-wide)

This file gives coding assistants and contributors the project-wide rules for building **Gait**.

## What Gait is

Gait is the offline-first policy-as-code runtime for AI agent tool calls. It bootstraps repo policy with `gait init` and `gait check`, captures signed evidence, and enforces fail-closed policy at the tool boundary.

Core primitives:

- **Jobs**: dispatch multi-step agent work with checkpoints, pause/resume/cancel, approval gates, and deterministic stop reasons (`gait job submit/status/checkpoint/pause/resume/cancel/approve/inspect`)
- **Packs**: unified portable artifact envelope (PackSpec v1) for run, job, and call evidence with Ed25519 signatures and SHA-256 manifest (`gait pack verify/diff`)
- **Gate**: evaluate structured tool-call intent against YAML policy with fail-closed enforcement (`gait gate eval`)
- **Regress**: convert incidents into deterministic CI regression fixtures with JUnit output (`gait capture`, `gait regress add`, `gait regress bootstrap`)
- **Voice**: gate high-stakes spoken commitments before utterance via signed SayToken capability tokens and callpack artifacts (`gait voice token mint/verify`, `gait voice pack build/verify/diff`)
- **Context Evidence**: deterministic proof of what context the model was working from, with privacy-aware envelopes and fail-closed enforcement when evidence is missing
- **Doctor**: first-5-minutes diagnostics (stable JSON + fixes) (`gait doctor --json`)

Supporting surfaces:

- **Registry**: signed/pinned skill pack install + verify workflows
- **MCP proxy/bridge/serve**: transport-aware boundary adapters (`stdio`, `SSE`, streamable HTTP) that route through Gate policy evaluation
- **Scout**: local snapshot/diff/signal reporting for drift and incident clustering

The durable product contract is **artifacts and schemas**, not a hosted UI.

## Non-negotiable contracts

- **Determinism**: `verify`, `diff`, and **stub replay** must be deterministic given the same artifacts.
- **Offline-first**: core workflows must not require network access.
- **Default privacy**: record reference receipts by default (no raw sensitive content unless explicitly enabled).
- **Fail-closed safety**: in “production/high-risk” modes, inability to evaluate policy blocks execution.
- **Schema stability**: artifacts and `--json` outputs are versioned and remain backward-compatible within a major.
- **Stable exit codes**: treat exit codes as API surface; add new codes only intentionally.

## Architecture boundaries

- **Go is authoritative** for: schemas, canonicalization, hashing, signing/verification, zip packaging, diffing, stub replay, policy evaluation, job lifecycle, voice token minting/verification, and CLI output.
- **Python is an adoption layer only**: capture intent, call local Go, return structured results. No policy parsing/logic in Python. Keep SDK ergonomics thin (`ToolAdapter`, minimal decorators), not framework replacement.
- **Wrappers and sidecars are transport only**: all enforce/allow/block decisions come from Go (`gait gate eval`, `gait mcp proxy`, `gait mcp serve`), never framework-local logic.
- **Next.js docs site** (`docs-site/`): static export deployed to GitHub Pages. Reads markdown from `docs/` via gray-matter frontmatter. No runtime dependencies.
- **Node/TypeScript**: limited to docs site and local UI shell (`ui/local/`). Not part of the core CLI path.

Current reference adapter set (keep parity): `openai_agents`, `langchain`, `autogen`, `openclaw`, `autogpt`, `gastown`, `voice_reference`, and the canonical sidecar path.

## Canonicalization, hashing, and artifacts

- Any JSON that participates in a digest, signature, cache key, or diff MUST be canonicalized using **RFC 8785 (JCS)** before hashing/signing.
- Zip artifacts must be **byte-stable** when regenerated from identical inputs:
  - deterministic file ordering
  - stable timestamps (fixed epoch)
  - stable file modes/ownership metadata
  - explicit compression settings
- Never hash “pretty printed” JSON or platform-dependent encodings.

## Security and privacy

- Never commit secrets, tokens, private keys, or real customer data.
- Avoid logging sensitive payloads; prefer digests + redaction metadata.
- All “unsafe” operations (real tool replay, raw capture, destructive tools) require explicit flags and must be obvious in help text and JSON outputs.
- Use standard crypto primitives (ed25519, sha256) from well-reviewed libraries; no custom crypto.

## Engineering standards

### Go

- Format: `gofmt` always; keep code idiomatic and boring.
- Errors: wrap with `%w`; return typed sentinel errors only when they improve caller behavior.
- Concurrency: keep it explicit; no background goroutines without lifecycle control.
- Time/locale: avoid locale-dependent formatting; timestamps should be RFC 3339/UTC or fixed epochs as defined by schema.
- IO: be careful with filesystem permissions; artifacts should be readable by the user but not world-writable by default.

### Python (wrapper SDK)

- Keep Python “thin”: serialization, subprocess/FFI boundary, and ergonomics only.
- Prefer strict typing; keep the public wrapper API small and stable.
- Tooling targets: `uv`, `ruff`, `mypy`, `pytest`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clyra-AI/gait](https://github.com/Clyra-AI/gait) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
