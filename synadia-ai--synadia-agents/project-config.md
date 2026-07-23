---
trigger: always_on
description: This file orients Claude Code when working anywhere in this repo. For
---

# CLAUDE.md

This file orients Claude Code when working anywhere in this repo. For
language-specific deep dives, defer to the package-level CLAUDE.md (e.g.
`client-sdk/python/CLAUDE.md`) and to per-package READMEs.

## What this repo is

The home for everything around the **Synadia Agent Protocol for NATS** — language
SDKs that callers use, agent plugins that host the protocol, and example
apps that exercise both. The root `README.md` covers the user-facing
view: layout, subject namespace, wire shape, quickstart. Read it before
making changes that touch user-visible surface.

The protocol spec is **not vendored here**. Canonical source:
[`synadia-ai/synadia-agent-sdk-docs`](https://github.com/synadia-ai/synadia-agent-sdk-docs)
— always link to the GitHub URL when reasoning about wire shape. Local
`docs/protocol-mapping.md` files inside each SDK translate spec → impl;
they are not the spec itself.

## Reference agents — canonical implementations

Every SDK in this repo ships a spec-compliant **reference agent** that
implements the full §12 agent-checklist (service registration, prompt
endpoint, status endpoint, heartbeats on `agents.hb.*.*.*`, terminator
semantics). When reasoning about wire shape, expected on-the-wire
behaviour, or testing a new SDK / agent, read these first — they are
the authoritative on-the-wire counterpart to the spec doc.

- **TypeScript**: `agent-sdk/typescript/src/testing/reference-agent.ts`
  — the `ReferenceAgent` class, importable from third-party packages
  via `@synadia-ai/agent-service/testing` (post-SDK-split — pre-split
  it lived under `@synadia-ai/agents/testing`). Runnable script:
  `client-sdk/typescript/examples/_run-reference-agent.ts`.
- **Python**: `agent-sdk/python/examples/_reference_agent.py` — a
  runnable echo agent with conversation memory, used as the test
  harness for the numbered demos and as a wire-compat counterparty.
  (Moved with the host-side split into `synadia-ai-agent-service`; pre-
  split it lived under `client-sdk/python/examples/`.)

Each SDK also has a parallel set of numbered demo scripts that exercise
discovery, prompting (text + attachments), mid-stream queries, and
liveness against the reference agent — useful both as documentation
and as a smoke surface:

- TS: `client-sdk/typescript/examples/01-discover.ts` … `05-liveness.ts`,
  plus `06-chat.ts` (interactive REPL).
- Python: `client-sdk/python/examples/01-discover.py` … `05-liveness.py`,
  plus `06-chat.py` (interactive REPL). See
  `client-sdk/python/examples/README.md` for the full table.

The Python interop test (`client-sdk/python/tests/test_interop_e2e.py`)
runs the TS reference agent as a subprocess and validates wire
compatibility between the two SDKs.

## Repository layout (and what's published)

| Path | Package | Published as | Notes |
| --- | --- | --- | --- |
| `client-sdk/typescript/` | `@synadia-ai/agents` | npm (public) | TS SDK — Node/Bun callers |
| `client-sdk/python/` | `synadia-ai-agents` (import: `synadia_ai.agents`) | PyPI | Python SDK — has its own CLAUDE.md |
| `agents/pi/` | `@synadia-ai/nats-pi-channel` | npm (public) | PI extension plugin |
| `agents/openclaw/` | `@synadia-ai/nats-channel` | npm (public) | OpenClaw plugin |
| `agents/claude-code/` | `claude-channel-nats` | npm (public) | Claude Code MCP plugin |
| `agents/hermes/` | — | not in repo | README only; ships from upstream Hermes |
| `agents/flue/` | `@synadia-ai/flue-nats-channel` | npm (public) | Flue sidecar channel |
| `agents/opencode/` | `@synadia-ai/opencode-nats-channel` | npm (public) | OpenCode plugin channel |
| `agents/codex/` | `@synadia-ai/codex-nats-channel` | npm (public) | Codex app-server-backed channel |
| `agents/acp/` | `@synadia-ai/acp-nats-channel` | npm (public) | Generic ACP channel (grok preset + custom) |
| `agents/grok/` | `@synadia-ai/grok-nats-channel` | npm (public) | Grok Build front door — thin pin over `agents/acp` |
| `examples/pi-headless/` | `@synadia-ai/nats-pi-headless` | npm (public) | depends on `@synadia-ai/agents@^0.5.x` |
| `examples/agent-web-ui/` | `@synadia-ai/nats-ai-testui` | github only | local-clone test client; `private: true` so it never publishes |
| `examples/dspy/` | `@synadia-ai/nats-dspy-agent` | private | uses `file:` link to local SDK |

**No root `package.json`, no workspace manager.** Each subtree manages
its own deps and tooling. Examples that ship to npm pin the SDK by
semver range (`^0.1.x`); private/dev-only examples (currently just
`dspy`) use `file:../../client-sdk/typescript`.

**Agents _should_ reuse the SDK package where it helps.** The SDK
(primarily the TS one) has accumulated meaningful shared logic —
framing, validation, helpers — and the direction of travel is an SDK
that covers **both client and agent sides** of the protocol. If an
agent harness needs functionality the SDK already provides, prefer
adding `@synadia-ai/agents` to its `package.json` and importing from
there over inlining or copy-pasting.

This is a reversal of the prior rule that agents must stay on raw
`@nats-io/*`. That rule no longer applies — duplicated hand-written
code in the agent harnesses is now technical debt, not a deliberate
boundary.

Caveat: this is a forward-looking rule, not a retroactive sweep. The

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synadia-ai/synadia-agents](https://github.com/synadia-ai/synadia-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
