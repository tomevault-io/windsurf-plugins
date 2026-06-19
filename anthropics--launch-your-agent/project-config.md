---
trigger: always_on
description: <!-- Copyright 2026 Anthropic PBC -->
---

<!-- Copyright 2026 Anthropic PBC -->
<!-- SPDX-License-Identifier: Apache-2.0 -->

# cma-test — "launch your managed agent" skill

## What this project is
A Claude Code **skill** (`launch-your-agent`) that helps a **technical founder** build whatever they want on **Claude Managed Agents (CMA)** — an internal worker, a piece of their product, a customer-facing agent; recurring or one-off: interview → build kit → launch → eval → iterate → scheduled deployment (when it should run on a clock). General-purpose, not a workshop — no clock/"hour" framing, no "recurring task" requirement; the spine is an iterative lens (scope a v0, launch and grade it, layer upgrades as v1/v2 via NEXT-DIRECTIONS).

## Repo map (state as of 2026-06-15)
| Path | What it is |
|---|---|
| `cma-primitives.md` | Full inventory of CMA primitives/limits from the live docs (incl. **scheduled deployments** at `/managed-agents/scheduled-deployments`) |
| `.claude/skills/launch-your-agent/references/examples-bank.md` | **Sourced-only** archetypes (each traced to an official cookbook/doc), production proof-point links, §5 = other-ideas backlog |
| `ui/agent-overview-example.html` + `ui/build-sheet.example.json` | The overview page example (live-schema dashboard, kept identical to the skill's `overview-template.html`) + example build sheet — open locally in a browser |
| `.claude/skills/launch-your-agent/` | **The skill**: SKILL.md (4 phases) + references/ (interview, cma-api, examples-bank, overview-template, build-sheet example). Also installed at `~/.claude/skills/launch-your-agent/` for testing — keep both copies in sync (repo is the source of truth). |
| `.claude/skills/wrap-up/` | **Companion skill** `/wrap-up`: owns the close-out / status check for a built agent (overview regen, primitives recap, run log, tailored extensions, hygiene sweep). Reuses launch-your-agent's references via `../launch-your-agent/references/`. Installed at `~/.claude/skills/wrap-up/`. launch-your-agent's Phase 4 invokes it. |

## Key decisions
- Audience: technical founder, self-directed, inside Claude Code. Show the machinery (curl/`ant`), don't hide it.
- **Build what they need, scoped into versions** (not "defaults until earned"): starting point is one agent, **Opus-class by default** (Sonnet when speed/cost matters more for the use case), full toolset, cloud env, **Outcome kickoff** (`user.define_outcome`, max_iterations 3), drafts-only — but any primitive (connector, memory, custom tool/skill, schedule) goes in v0 if the core job needs it and it's wireable now; everything else is laid out as numbered v1/v2 increments in NEXT-DIRECTIONS.
- **Open light, boundaries in context**: the opening is a couple of warm sentences + 2–3 archetype examples + the open question — no version vocabulary, no boundaries block, no process talk. Caveats/guardrails are raised briefly at the moment they matter. After the founder's first answer, **let them explain** (one open follow-up) before any suggestions, reshaping, or AskUserQuestion menus. Then one cluster at a time, AskUserQuestion for enumerable choices, max one open-ended question per turn.
- **Don't undersell**: keep hard limits (sub-second latency, live phone calls) separate from defaults — building a UI on top of the agent is fine (that's the generated-interface extension, not a limitation); gated write-actions (`always_ask`, paper/sandbox variants) are possible even in v0 if the founder wants them; drafts-first is the recommendation, not a "never".
- **No spend-limit step** — the iteration bound (`max_iterations: 3`) is the only quiet cost default; mention runs cost cents, nothing required in the Console.
- **Never stop to wait for the key — heads-up early, hand over late**: as soon as a credential is inevitable (API key, connector token/webhook), tell the founder where to get it (a "grab these while I stage" table rides with the brief) so they fetch in parallel; stage everything keyless meanwhile. Easiest handover: **check the shell env first** (many founders will already have it exported — skip the ask if so); otherwise pre-create `my-agent/.env` (chmod 600, gitignored) and the founder either pastes the key into it via its absolute path or exports it in their own terminal — whichever is quicker for them. Only hard rule: never into chat. Key that touches a transcript ⇒ rotate.
- **Connectors are named, not hidden — and mockable**: when delivery is asked for (email/Slack/etc.), state the guardrail (drafts-only + `always_ask` gate) and the connector route. Slack and email both take real setup (app/OAuth/webhook + vault), so the **default is mock-in-v0, wire-for-real in v1**; if the founder wants to wire it now, do it — pull the latest connector docs first. Don't underbuild: if the core job needs a connector (credential on hand) or a memory store, it goes in v0, not deferred reflexively. Mocking = outbox of schema-true payloads or stubbed custom tool — `references/mock-connectors.md` has the patterns + typical endpoint schemas — so v1 is just the swap to the real MCP server + vault.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthropics/launch-your-agent](https://github.com/anthropics/launch-your-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
