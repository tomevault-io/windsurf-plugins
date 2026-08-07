---
trigger: always_on
description: <!-- Modified 2026 by Sereja Ris for VibecodersCrew (community fork of Kiro Crew). See NOTICE and CHANGELOG.md. -->
---

<!-- Modified 2026 by Sereja Ris for VibecodersCrew (community fork of Kiro Crew). See NOTICE and CHANGELOG.md. -->
# Rules for AI Assistants

This is the **single source of truth** for working in this repository (the
frontend has its own `website/AGENTS.md`). Read it before non-trivial changes.

## What this is

This checkout is the unofficial `serejaris/vibecoderscrew` community fork.
Its default branch is `codex-main`. Releases are source-only previews, all
telemetry is hard-disabled, and the Amazon update/signing/CDN lanes are outside
this fork's release boundary.

KiroCrew is an open-source personal AI agent that runs on your own machine —
chat from Slack, a web dashboard, or the CLI; run multi-step tasks unattended;
schedule cron jobs; persist memory across sessions. It drives an LLM through
Kiro ACP (`kiro-cli`) or the official OpenAI Codex App Server, plus MCP tools.

- **Backend:** Python package `kiro_crew` in `src/kiro_crew/`.
- **Frontend:** React + TS + Vite SPA in `website/`; built `dist/` is staged into
  `src/kiro_crew/static/dist/` and served by the backend.
- **Distribution:** public GitHub → `pip install` (backend) + `npm`/Vite
  (frontend). Plain setuptools — **no Brazil, no internal build tooling.**
- **Data home:** `~/.kiro/crew` (nested under kiro-cli's `~/.kiro/`); the legacy
  `~/.kirocrew` is auto-migrated. Override with `KIROCREW_HOME`.

## This is a public OSS fork — do not re-introduce internal couplings

This repo is the de-Amazoned public fork of an internal package. When adding or
changing code, **never reintroduce** any of the following:

- Build/infra: Brazil (`Config`, `AUTOSDE.yaml`, `CODE_APPROVERS.yaml`),
  `npm-pretty-much`, toolbox bundler, AIM hooks, CodeArtifact registries. Use
  setuptools + public PyPI / public npm only.
- Services/auth: enterprise SSO, MCS, Kerberos, federated login, device-posture
  tunnels, Cognito/RUM ids, builder-mcp, `arcc`, Quip, internal ticketing. The
  internal marker names have been scrubbed from code, comments, and docs — do not
  reintroduce them. `scripts/scrub-lint.sh` gates the scanned source roots
  (`src/`, `website/src/`, `scripts/`, `config/`, `packaging/`, top-level); `docs/`
  is allowlisted (it legitimately describes platform-seam / SSO-marker concepts)
  so keep docs clean by convention.
- These subsystems are **stubbed** (`sso_status.py`, `browser/auth.py`,
  `dashboard/handlers/sso_login.py`, `tunnel/manager.py`, `aim_agents.py`): their
  public symbols are preserved as no-ops so the import graph stays intact — keep
  them stubbed.
- KiroCrew supports two public providers: `acp` drives `kiro-cli`, and `codex`
  drives the official OpenAI Codex App Server using the user's existing Codex
  login. Keep provider selection, native session resume, model discovery,
  approvals, token usage, and prerequisite gates working for both. The
  standalone `ClaudeCodeProvider`/`BedrockProvider`/`cc_agent`/`mirror` modules,
  the `claude_code`/`bedrock` factory branches, the `cc_*`/`bedrock_*` config
  fields, and the `[aws]` extra are gone. The dormant `ACP_BACKEND_CLAUDE` /
  `_is_claude` seam in `acp/client.py` is intentionally kept so an internal
  companion can re-register Claude Code — do NOT delete it, and do NOT re-add
  the removed standalone Claude/Bedrock registration glue.
- OSS-flipped defaults (keep): embeddings are **always-on and in-process**
  (vendored llama-cpp-python under `_vendor/`; Qwen3 GGUF over sha256-pinned HTTPS
  from the KiroCrew CDN, override via `KIROCREW_EMBED_MODEL_URL` /
  `memory.embed_model_url`; the `EmbeddingBackend` seam keeps other runtimes
  possible); voice TTS defaults to **Piper** (local); Slack enterprise gate is
  default-open (opt-in allowlist via `slack.allowed_enterprise_ids`); `boto3` /
  `amazon-transcribe` are optional lazy imports for STT (`pip install
  kirocrew[voice]`).

**Keep** the generic security controls (not internal-specific): AKIA/ASIA
credential redaction, destructive-command deny patterns, `~/.aws` / `~/.ssh`
sensitive-path blocking, SEL audit log. The deny patterns are first-class
`DeniedCommandRule` records (`BUILTIN_DENIED_RULES`, **137 rules**) enforced only
at the `hooks.py` PreToolUse gate; default-ON but user-configurable from Settings
→ Security, with the governance `commands` scope as the un-opt-out-able enterprise
force-pin. See `docs/system-specs/modules/security.md`.

**Fork-initiated UX divergences (do not let an upstream sync re-introduce):** the
**Channels** app is hidden from the App Store, the **Board** app is removed, and
the Voice panel adds a local **Piper** TTS provider. These are launch product
choices tracked out-of-tree with the upstream sync tooling.

The artifact **Iterate** divergence is retired: `SHOW_ARTIFACT_ITERATE` gated the
old navigate-away flow "pending an artifact redesign", and the embedded companion
chat panel IS that redesign — so the flag is deleted and the affordance ships. Do
NOT re-add the gate. See `docs/system-specs/modules/artifacts.md` § Companion Chat.

## Platform layer: CPP seam + Governance (read before touching `platform/`)

`src/kiro_crew/platform/` is the **Composed Platform Providers (CPP)** edition

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [serejaris/vibecoderscrew](https://github.com/serejaris/vibecoderscrew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
