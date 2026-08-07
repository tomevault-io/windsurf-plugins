---
trigger: always_on
description: Orientation for Claude Code (and any new contributor) working in this repo.
---

# CLAUDE.md

Orientation for Claude Code (and any new contributor) working in this repo.
Read this first; it assumes you know nothing about the project.

## What this repo is

**`synadia-agents-labs`** is standalone **courseware** — small, runnable,
heavily-commented examples for the **Synadia Agent Protocol for NATS** SDKs. It
accompanies Synadia workshops and trainings, and doubles as a self-study ground.

It is deliberately **independent** of the SDK *source*: every example depends on
the **published** packages (`@synadia-ai/agents` + `@synadia-ai/agent-service`
from npm; `synadia-ai-agents` + `synadia-ai-agent-service` from PyPI). There is
**no link back** to the SDK monorepo — no `file:` deps, no submodules, no
relative imports into another repo. You can develop here without the SDK source
checked out.

- **User-facing overview:** [`README.md`](README.md) — layout, conventions, the
  environment-variable reference, quickstart. Keep it in sync when you change
  user-visible surface.
- **The protocol spec** (source of truth for wire shape) is **not vendored**:
  <https://github.com/synadia-ai/synadia-agent-sdk-docs>. Link to it; don't copy
  it.
- **The SDK source** lives in the separate `synadia-ai/synadia-agents` monorepo.
  It's a *reference*, not a dependency. (On this machine it's typically a sibling
  dir: `../synadia-agents`.)

## The protocol in 30 seconds

An **agent** is a NATS micro-service identified by a triple
`<agent>/<owner>/<session>` (e.g. `echo/alice/main`) — the SDK calls the 5th
subject token `name` in TypeScript and `session_name` in Python. From it the SDK
derives subjects:

- `agents.prompt.<agent>.<owner>.<name>` — send a prompt, stream a reply
- `agents.status.<agent>.<owner>.<name>` — status endpoint
- `agents.hb.<agent>.<owner>.<name>` — heartbeats (liveness)

A reply is a stream of typed frames: `{"type":"status","data":"ack"}` →
`{"type":"response","data":"…"}` (one or many) → an **empty terminator**.
**Callers** (`@synadia-ai/agents`) discover agents and prompt them; **agents**
(`@synadia-ai/agent-service`) host the endpoints. Wire **protocol version is
`0.3`**; the published package version is currently `0.5.2` (different axes —
don't conflate).

## Layout

```
typescript/
  client/   01-discover 02-prompt-text 03-prompt-attachment 04-query-reply 05-liveness 06-chat
  agent/    01-echo 02-llm-ollama 03-llm-openrouter 04-llm-combined 05-system-prompt 06-chat-agent reference-agent
python/     parity with the TypeScript examples through agent 04 (agent 05–06 not yet ported)
projects/
  tool-calling/  typescript/ + python/ agents (01-single-tool … 04-memory) +
                 responders/ (calculator·time·temperature·inventory·memory, polyglot;
                 the calculators join queue group "calculators" → fleet demo)
  human-tool/      TS agent whose ask_expert tool is answered by a HUMAN (`nats reply tools.expert.<owner>`)
  crowd-poll/      TS agent whose ask_the_room tool scatter-gathers `room.poll` via requestMany
  agent-of-agents/ TS "reporter" agent whose tools are the caller SDK (discover + prompt other agents)
  edge-sensor/   cross-domain capstone (TS agent + Go bridge + Go mock sensor)
labs/       guided workshop track — numbered lab sheets (00-setup … 09-agent-of-agents),
            plus `-python` twins (00/02/04/05) where the repo has Python parity.
            TRACKED courseware: attendees follow these from their checkout; secrets
            (creds, API keys) are never in them — those go via workshop chat
tests/      verification harness — runs every example, writes a transcript (tests/README.md)
infra/      dev container + docker-compose (nats-server + Node·Bun·Python·uv·Go + nats CLI; no LLM bundled) — see infra/README.md
presenter/  gitignored, presenter-only: live runbook + workshop structure (not courseware)
docs/       (planned — protocol primer & deeper guides)
```

Two tiers: the **core examples** (small, single-language lessons under
`typescript/` & `python/`, split into `client/` callers and `agent/` hosts) and
**`projects/`** (bigger, self-contained demos, sometimes multi-language — e.g.
tool-calling with polyglot responders, the edge-sensor capstone).
Terminology: call the numbered sequences the **client/agent examples** — the
word "ladder" is banned in all text (maintainer preference).

## Conventions — treat these as rules

1. **Every example is self-contained.** Its own `package.json` (TS) /
   `pyproject.toml` (Py), its own `tsconfig.json`, its own README, all of its
   code in the folder. You can copy a single example folder out of the repo and
   it still runs. We **accept duplication** as the price of clarity — a learner
   should see *exactly* what an example uses without chasing imports.
2. **Pinned, published SDKs.** Depend on the published packages by semver, never
   a path back to the SDK source: npm `@synadia-ai/agents` + `@synadia-ai/agent-service`
   (`^0.5.2`), PyPI `synadia-ai-agents` (`>=0.7`) + `synadia-ai-agent-service`
   (`>=0.4`). To bump, edit each example's `package.json` / `pyproject.toml`
   (there is no shared manifest by design).
3. **Role-based filenames.** The runnable file is named for its protocol role:
   `agent.{ts,py}` (a host), `client.{ts,py}` (a caller), `responder.{ts,py}` (a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synadia-ai/synadia-agents-labs](https://github.com/synadia-ai/synadia-agents-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
