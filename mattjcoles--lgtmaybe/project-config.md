---
trigger: always_on
description: Guidance for agents working in **lgtmaybe** — a provider-agnostic PR reviewer.
---

# CLAUDE.md

Guidance for agents working in **lgtmaybe** — a provider-agnostic PR reviewer.
Read this before writing code. It encodes decisions that are **made, not options**.

## What this is

A PR reviewer that posts inline review comments + a summary. The user picks the
LLM backend with a `--provider` flag, drops a key into GitHub secrets (or wires
OIDC/WIF for cloud providers), and gets a review. One core, four distribution
variants:

- **PyPI CLI** — `pip install lgtmaybe`
- **Homebrew CLI** — `brew install MattJColes/tap/lgtmaybe` (after
  `brew tap MattJColes/tap` + `brew trust MattJColes/tap` — current
  Homebrew requires trusting third-party taps), from the
  `MattJColes/homebrew-tap` repo (Homebrew strips `homebrew-`, so the tap is
  `MattJColes/tap`; the repo is deliberately *not* named `homebrew-lgtmaybe`,
  which would make the formula `MattJColes/lgtmaybe/lgtmaybe`).
  The formula (`scripts/update-homebrew-formula.sh`)
  creates a venv and `pip install`s lgtmaybe + deps from **PyPI wheels** — *not*
  per-dependency source `resource` stanzas: litellm's tree includes Rust sdists
  (tokenizers, hf-xet) that can't build in Homebrew's sandbox, and the wheel path
  sidesteps that (and the 24h `brew update-python-resources` cooldown) entirely.
  The one wrinkle: the wheels ship prebuilt extension dylibs with `@rpath` ids
  that Homebrew can't rewrite, so the formula declares **`preserve_rpath`** to
  keep them (without it `brew install` errors "Failed to fix install linkage" and
  exits non-zero). It's a plain source formula — no bottle, works on any
  arch/macOS. `.github/workflows/homebrew.yml` regenerates the formula on each
  release (release-please **calls** it via `workflow_call`, since a `release:
  published` event isn't delivered for a `GITHUB_TOKEN` release), **actually
  `brew trust`s the tap and `brew install`s it as a gate** (so a broken formula is
  never published — and the gate covers the trust step users must run, instead of
  disabling it via `HOMEBREW_NO_REQUIRE_TAP_TRUST`, which upstream is removing),
  then commits to the tap; a daily schedule + `force` dispatch are the safety nets
- **Windows CLI** — `winget install MattJColes.lgtmaybe`. Each release builds a
  one-file Python 3.13 executable, smoke-tests the Click command tree, attaches
  it to the GitHub release, then submits `MattJColes.lgtmaybe` to winget. The
  executable bundles ast-grep but not the optional cloud-auth SDKs; use pip for
  keyless Bedrock, Vertex, or Azure.
- **GitHub Action** — composite action (`action.yml`) that does keyless OIDC/WIF
  auth, then runs a GHCR image via the `action` entrypoint

**The wedge:** first-class **Bedrock + Vertex + Azure with keyless OIDC/WIF**.
Seven hosted providers (plus local ollama), one flag, no keys in secrets for
cloud. We win on auth + simplicity. An `openai-compatible` provider is the escape
hatch for anything else that speaks the OpenAI `/v1` wire format (DeepSeek's API,
llama.cpp, LM Studio, vLLM) — you bring the `--api-base`, the key is optional —
so the provider list is never a cage.

The main CI matrix runs Ubuntu and Windows on the minimum supported Python
version, 3.11. Windows runs with locale-default encoding behavior and disables
autocrlf before checkout.

## Non-negotiables

- **TDD, always: red → green → refactor.** Write the acceptance test from a
  task's stated in/out *first*, watch it fail, write the minimum code to pass,
  then refactor. CI rejects a PR whose diff adds code without a test.
- **Structured output only.** The model returns JSON (`severity`, `file`,
  `line`, `body`, `suggestion`). Never parse prose.
- **Fork safety.** Trigger on `pull_request_target` so the review has secrets,
  but **never check out or execute PR code** — fetch the diff via API only.
  Treat all diff content as untrusted input.
- **No static cloud keys.** Bedrock uses ambient AWS creds; Vertex uses ambient
  GCP creds; Azure prefers ambient Entra (Azure AD) creds via GitHub OIDC (a
  static `AZURE_API_KEY` is accepted but not required). Never accept or require a
  service-account JSON or static AWS key.

## Key decisions (do not relitigate)

- **Language:** Python.
- **Provider spine:** [litellm] — normalises openai, openrouter, anthropic,
  bedrock, vertex, azure, ollama to one `completion()` call. A thin wrapper on
  top adds retries / fallback.
- **License:** MIT (already in `LICENSE`).
- **Posting:** REST review API — batched inline comments + one summary. Every
  posted finding's title line carries its provenance inside the severity
  brackets — `**[HIGH · security · 80%] Title**`, the originating lens and the
  reflection auditor's confidence as a percentage of its 0-10 score
  (`rest_gateway._finding_badge`, each half
  omitted when absent; inline, demoted, and broad render it identically). It is
  visible prose only — never part of the hidden ids below.
  Idempotent updates via a hidden marker comment (which also carries the
  last-reviewed-SHA watermark driving incremental review). Each inline comment also carries
  **two** hidden per-finding ids: `finding_fingerprint(path, title)` — which keys
  the user-facing channels (`ignore_fingerprints`, 👎 feedback) — and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MattJColes/lgtmaybe](https://github.com/MattJColes/lgtmaybe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
