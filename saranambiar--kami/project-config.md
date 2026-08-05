---
trigger: always_on
description: This is the guiding document for the coding agent building this project. Read fully before starting. Public setup lives in `README.md` / `SETUP.md`. Optional local-only notes (`tasks/`, `lessons/`, `memory/`) may exist on your machine but are not part of the published Community Edition tree.
---

# AGENTS.md — GTM-as-a-Service Agent Org (Hermes Buildathon)

This is the guiding document for the coding agent building this project. Read fully before starting. Public setup lives in `README.md` / `SETUP.md`. Optional local-only notes (`tasks/`, `lessons/`, `memory/`) may exist on your machine but are not part of the published Community Edition tree.

---

## 1. What we're building

An open-source, multi-agent marketing/GTM agency — powered by real running **Hermes agents** (Nous Research's agent framework), not a custom app that merely calls an LLM API. Hermes itself is the backend. We are not simulating agents; Hermes processes are the manager and the specialists.

**Team:** 2 people, 8-hour build sprint, at the Hermes Buildathon, "AI as Agency" track.

**Core mechanic:** A user submits a campaign brief (company, ICP, goal, tone) through a web app on our own domain. That request hits a persistent Hermes manager session running on our server. The manager plans, delegates to specialist Hermes subagents, reviews their output, and the specialists execute real actions on real surfaces (send a real email, publish a real post) — not staged or sandboxed.

**Capabilities (specialists), in priority order — do not build all of these to equal depth:**
1. **Outreach engine (one build, three playbooks)** — outbound/SDR, investor/VC, influencer outreach. Same pipeline: research target → apply the relevant playbook → send real email → log. Build this once, well, and parametrize the playbook.
2. **Content specialist** — research an angle/trend → draft → publish to a real platform (X, plain text posts).
3. **Manager agent** — parses the brief, produces a genuinely different plan per genuinely different brief, delegates, reviews specialist output, bounces at least one thing back for revision before it ships.

**Explicitly descoped for the 8-hour build (do not attempt unless everything above is done early and solid):** Community engagement (Reddit/Discord), LinkedIn company-page posting (requires multi-week API approval — do not attempt). If time allows, the flex move is the manager **spawning a new specialist role live** (e.g. a "PR angle" specialist) rather than half-building a fifth pre-built specialist.

**Scope discipline:** We will not spend the full 8 hours purely coding. Time is also needed for shooting demo/content footage and rehearsing the live demo. Budget accordingly — treat roughly the back 90 minutes as reserved for demo prep, not new features, regardless of how development is going.

---

## 2. Architecture — how this actually gets deployed and accessed

**Non-negotiable requirement:** Judges access this via a real link to our own domain, from their own device, with no install. Do not build a Telegram-bot-only flow as the primary surface (a bot can be an *additional* nice-to-have, not the main path).

**How it works:**
```
judge's browser → yourdomain.com (TLS) → reverse proxy → Hermes API Server (running on our VPS)
                                                              → manager session (role=orchestrator)
                                                              → delegate_task → specialist subagents
                                                              → real email send / real post / real research
```

- **Host:** a small always-on VPS (not a laptop — must survive the whole judging window). Set this up first, before feature work.
- **Backend:** Hermes running with the `api_server` platform enabled in `config.yaml`, exposing an OpenAI-compatible `/v1/chat/completions` endpoint natively off the running gateway process. This is Hermes's built-in API Server adapter — do not build a custom bridge.
- **Domain:** our own purchased domain, A record pointed at the VPS, TLS via reverse proxy (Caddy is simplest — automatic HTTPS in ~5 lines of config).
- **Frontend:** a thin web app (chat UI or campaign-brief form — form is more demo-friendly if time allows) that POSTs to the Hermes API Server endpoint with a bearer token, and streams the manager's response. Each browser session gets its own `X-Hermes-Session-Id` so concurrent judges don't collide — Hermes supports multiple concurrent sessions natively.
- **The manager = a Hermes session running with `role="orchestrator"`.** It is a real Hermes agent instance, not a wrapper.
- **Specialists = subagents spawned via `delegate_task`**, isolated context/terminal/toolset per specialist, running inside the same Hermes process. Use the `async_delegation` toolset (non-blocking) so the manager can dispatch outreach + content in parallel without freezing the session — verify this is stable on install before relying on it for the demo.
- **Playbooks = Hermes skills.** Each GTM playbook (see Section 4) is a `SKILL.md` file under `~/.hermes/skills/`. Skills are shared automatically with spawned subagents — write playbooks as skills, not hardcoded prompts, so they're inspectable, versioned, and improvable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saranambiar/kami](https://github.com/saranambiar/kami) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
