---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in the **Agent37 Starter Kit**.
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in the **Agent37 Starter Kit**.
`CLAUDE.md` imports this file via `@AGENTS.md`, so this is the single source of
truth — edit here, not there.

## First-time setup

Setting this up from a fresh clone? Follow **[`SETUP.md`](SETUP.md)** — the complete runbook
(it's what the README tells adopters to hand you). Two login-gated secrets are human-supplied:
`AGENT37_API_KEY` (plus a **funded** Agent37 wallet) and `SUPABASE_ACCESS_TOKEN`;
`npm run setup` does the rest. Never print or commit the `sk_live_` key.

## What this project is

A full-stack starter for building your own agent app, built entirely on top of the
public **[Agent37](https://www.agent37.com) B2B Agents API**: email + password auth
(open signup, no verification), a multi-agent fleet, and — for each agent — native
in-dashboard **Chat**, a **Files** browser, **Integrations** (Composio), and a
**Settings** tab. Forkers rebrand it (`src/config/branding.ts`) and ship it; their
end users sign up, get workspaces, invite teammates, and create / manage agents.

Everything this app can do is a **subset of the Agent37 `/v1` API** — control plane
*and* data plane. This repo is a *client* of that API — it does not implement agent
infrastructure itself. So **the API docs, not this code, are the authority on what an
agent can and cannot do.**

## The API this is built on — read the docs first

This product is built on top of our public API. **Before adding or changing any
agent capability, consult the docs** — they define the full surface and its
limits. Two machine-readable entry points are designed for you (an AI agent) to
fetch directly:

- **<https://www.agent37.com/docs/llms.txt>** — concise index of every doc page.
  *Start here* to find the right page.
- **<https://www.agent37.com/docs/llms-full.txt>** — the entire documentation
  inlined into one file. Use for deep reference.
- Human-browsable docs: **<https://www.agent37.com/docs>**
  (append `.md` to any page URL to get raw markdown.)

### Documented capability map

Two planes, one `sk_live_` key — and this template now drives **both**. The
**control plane** manages instances (and the per-agent Composio integrations); the
**data plane** powers the native Chat and Files tabs.

**Control plane — `https://api.agent37.com/v1/*`** (the `sk_live_` key this app holds):

| Page | Covers | Used here |
|---|---|---|
| [Core concepts](https://www.agent37.com/docs/agents-api/concepts) | the model, auth, the two planes | read first |
| [Instances](https://www.agent37.com/docs/agents-api/instances) | create / list / get / start / stop / restart / update / resize / delete | ✅ |
| [Instance URLs](https://www.agent37.com/docs/agents-api/urls) | short-lived signed URLs to open an agent's ports | ✅ |
| [Templates](https://www.agent37.com/docs/agents-api/templates) | the agent images you can provision | ✅ |
| [Managed services & budgets](https://www.agent37.com/docs/agents-api/budgets) | per-agent managed-spend cap | ✅ |
| [Billing](https://www.agent37.com/docs/agents-api/billing) | wallet, compute prepay, usage | ✅ (usage) |
| [Run commands](https://www.agent37.com/docs/agents-api/exec) | exec a command inside an instance | available, not used |
| [Errors](https://www.agent37.com/docs/agents-api/errors) | machine-readable error codes | ✅ (mapped in `Agent37Error`) |

The **Integrations** tab is also control plane: it manages a per-agent Composio
entity through `/instances/{id}/integrations/*` (toolkits / connect / connections).

**Data plane — `https://{instanceId}.agent37.app/v1/*`** (talk to one agent's
gateway). Data-plane requests authenticate with the `X-Agent37-Key: sk_live_...`
header (raw key, no Bearer prefix; `Authorization` passes through to the app inside
the instance), while the control plane stays `Authorization: Bearer`. The native
**Chat** and **Files** tabs call these endpoints directly
(through this app's BFF). The signed-URL "open in new tab" shortcuts still exist
too — they just complement the in-dashboard UIs now rather than replace them:

| Page | Covers | Used here |
|---|---|---|
| [Send a message](https://www.agent37.com/docs/agents-api/chat) | post a message, get a response (`/v1/responses`) | ✅ (Chat) |
| [Streaming](https://www.agent37.com/docs/agents-api/streaming) | stream responses (SSE) | ✅ (Chat) |
| [Sessions & models](https://www.agent37.com/docs/agents-api/sessions) | conversation state, model selection | ✅ (Chat) |
| [Files](https://www.agent37.com/docs/agents-api/files) | list / read / write / archive files | ✅ (Files) |
| [Build a chat app](https://www.agent37.com/docs/agents-api/chat-app) | end-to-end guide for a chat UI | reference |

So: **what's possible** = the whole map above, and this template now exercises most
of it: the control-plane rows marked ✅, the native data-plane Chat and Files tabs,
the per-agent Integrations tab, *and* the signed-URL buttons that open each agent's
own dashboard / terminal / files UI in a new tab.

## How this app fits together

```
Browser ─▶ Next.js (this app) ─▶ control plane  https://api.agent37.com/v1   (instances, integrations)
   │            │              └▶ data plane     https://{instance}.agent37.app/v1   (chat, files)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent37-platform/starter-kit](https://github.com/agent37-platform/starter-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
