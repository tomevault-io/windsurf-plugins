---
trigger: always_on
description: Most agent frameworks and coding agents speak the **OpenAI API**. Because
---

# Using freellmpool as the free LLM backend for AI agents

Most agent frameworks and coding agents speak the **OpenAI API**. Because
`freellmpool proxy` *is* an OpenAI-compatible endpoint, you can point them at it
and they'll run on pooled free-tier inference — with failover when one provider
rate-limits you mid-run (exactly when long agent loops tend to die).

## Release status

- **Latest release: 0.13.0.** GitHub and PyPI both provide 0.13.0, including
  the Hermes profile, `freellmpool/spread` routing, public `/livez` and
  `/readyz`, authenticated `/v1/providers`, `/v1/models?ready=true`, refreshed
  providers, Vercel AI Gateway support, and OpenCode registry-readiness hardening.
  Install it with `python -m pip install freellmpool`.

- **Registry publication status: pending.** `opencode-freellmpool` and
  `opencode-freellmpool-tui` were not published on npm as of 2026-08-29; use
  their local-file install paths.

Start the gateway once:

```bash
freellmpool proxy --port 8080
export OPENAI_BASE_URL=http://localhost:8080/v1
export OPENAI_API_KEY=anything   # ignored by freellmpool
```

Then wire up your tool of choice.

Orchestrators can use `/livez` for liveness, `/readyz` for an
advisory local quota/cooldown snapshot, authenticated `/v1/providers` for
secret-free provider readiness, and `/v1/models?ready=true` for ready targets.
These endpoints do not probe upstream providers.

For structured setup, use profiles:

```bash
freellmpool profile list
freellmpool profile show opencode
freellmpool profile install opencode
freellmpool profile doctor opencode --dry-run
```

`freellmpool code <agent>` remains a compatibility shortcut that renders the
same profile quick-start. `profile install <agent>` is print-only and does not
edit third-party config files. The OpenCode plugin registers its provider and
six routing aliases through the runtime config hook without writing the user's
configuration.

## OpenAI Python SDK / OpenAI Agents SDK

```python
from openai import OpenAI

client = OpenAI()  # reads OPENAI_BASE_URL + OPENAI_API_KEY
resp = client.chat.completions.create(
    model="auto",  # let freellmpool pick the least-used free provider
    messages=[{"role": "user", "content": "Plan a 3-step refactor of foo.py"}],
)
print(resp.choices[0].message.content)
```

See [`examples/agent_openai_sdk.py`](../examples/agent_openai_sdk.py) for a
runnable version.

## Claude Code

Claude Code speaks the **Anthropic Messages API**, which freellmpool shims at
`/v1/messages` — so it can run on free models:

```bash
freellmpool proxy --port 8080
export ANTHROPIC_BASE_URL=http://localhost:8080
export ANTHROPIC_API_KEY=anything
claude   # now on free models
```

> Experimental. Text and tool-use (the agentic file-editing loop) are translated;
> text-only requests stream incrementally, while tool requests remain buffered.
> Vision isn't yet supported. Free models are weaker than Claude — great for
> cheap iteration, not a full replacement. `freellmpool code claude` prints this.

## OpenAI Codex CLI

Codex speaks the **Responses API**, which `freellmpool` shims at `/v1/responses`:

```bash
freellmpool proxy --port 8080
export OPENAI_BASE_URL=http://localhost:8080/v1
export OPENAI_API_KEY=anything
codex --config model_provider=openai   # or set base URL in ~/.codex/config.toml
```

> The Responses shim sends text-only events incrementally in protocol order.
> Tool-calling and richer Responses requests remain on the buffered compatibility
> path, so they do not claim incremental delivery.

## Hermes Agent (released in 0.12.0)

Hermes supports an OpenAI-compatible custom endpoint. The first-class profile
prints the supported config and never edits `~/.hermes/config.yaml`:

```bash
freellmpool profile install hermes
freellmpool profile doctor hermes --dry-run
# Interactive equivalent: hermes model → Custom endpoint → http://localhost:8080/v1
```

Use model alias `quality` in Hermes. For long OpenCode or other OpenAI-compatible
agent loops, start with `freellmpool/agent`: it stays in the strongest healthy
benchmark tier, then spreads quota and uses latency/health as tie-breakers.
Use `freellmpool/spread` when whole-pool breadth matters more than capability.

## Metaswarm external-tools review

Metaswarm can use `freellmpool` as a review-only external tool. The integration
is in [`integrations/metaswarm`](../integrations/metaswarm): copy
`freellmpool-review-adapter.sh` into `.metaswarm/adapters/freellmpool.sh`, then
add it to `.metaswarm/external-tools.yaml` with roles `review` and
`second_opinion`.

The adapter is deliberately not an implementer. It reviews a worktree diff
against a spec/rubric, runs a configurable strong-model panel through
`freellmpool`, and emits a metaswarm-style JSON envelope. If no strong provider
key is configured (`MISTRAL_API_KEY`, `NVIDIA_API_KEY`, or `OPENROUTER_API_KEY`
by default), it returns `error_type: "auth_missing"` before any provider call.

The first-class profile is:

```bash
freellmpool profile show metaswarm
freellmpool profile doctor metaswarm --dry-run
```

It documents one free/cheap worker lane through the local proxy, one larger
reviewer lane, Tailnet URL setup for remote agents, and Codex/Opus as explicit

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xzr/freellmpool](https://github.com/0xzr/freellmpool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
