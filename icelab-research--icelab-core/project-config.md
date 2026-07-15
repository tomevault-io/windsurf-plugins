---
trigger: always_on
description: Permanent instructions for every agentic coding session in this project. Read this file at the start of every session before doing anything.
---

# Repo rules

Permanent instructions for every agentic coding session in this project. Read this file at the start of every session before doing anything.

`CLAUDE.md` is a symlink to this file. Read this first; then read the `AGENTS.md` in the folder you're working in (it overrides anything here).

## Repo shape

- `apps/` — Elixir / Phoenix apps. Shared stack rules: [`apps/ELIXIR_RULES.md`](apps/ELIXIR_RULES.md). Shared decisions: [`apps/MEMORY.md`](apps/MEMORY.md).
- `core/` — sf-voice product (Rust API + React frontend). Multi-tenant for sf-voice's external customers; **not** the booking-app product.
- `infra/` — Docker compose, Caddy, GitHub Actions, droplet bootstrap.

Each folder has its own `AGENTS.md` and `MEMORY.md`.

### Monorepo + submodule topology

The parent repo is `git@github.com:sf-voice/sf-voice-core.git`. It is a
**polyglot workspace root** — three package managers share one checkout:

| Ecosystem | Manifest | Scope |
|-----------|----------|-------|
| Rust | `Cargo.toml` (workspace) | `core/backend/api`, `core/backend/entities` |
| JS/TS | `pnpm-workspace.yaml` | `core/frontend`, `core/*` |
| Elixir | `mix.exs` | `apps/ellie_ai`, `apps/resto_booking_app` |

**`core/` is a git submodule** (`git@github.com:sf-voice/core.git`, branch `main`).
This means:

- The parent repo pins `core/` to a specific commit. `git status` in the parent
  will show `m core` if the submodule's working tree has local changes.
- To commit work inside `core/`: cd into `core/`, commit there, then update the
  pin in the parent with a second commit.
- To pull the latest `core/` tip: `git submodule update --remote core`.
- Clone the whole thing: `git clone --recurse-submodules <parent-url>`.
  After a plain clone, run `git submodule update --init` to populate `core/`.

### Data-flow direction (allowed traffic between services)

```
core/frontend ──HTTP──▶ core/backend ──WS──▶ apps/ellie_ai
                                              │
                                              └──HTTP──▶ apps/resto_booking_app
```

- `core/frontend` talks only to `core/backend`. Never to ellie or resto.
- `core/backend` may call into `apps/ellie_ai` (VAD channel only, see `apps/ellie_ai/lib/ellie_ai_web/channels/vad_channel.ex`). Never to resto.
- `apps/ellie_ai` polls `apps/resto_booking_app` via the `RestoClient` module. Never the reverse.
- `apps/resto_booking_app` initiates no outbound HTTP. Exposes endpoints only.
- All service-to-service channels (HTTP and WS) auth with `INTERNAL_API_TOKEN`. New cross-service channels MUST use this token compared with `Plug.Crypto.secure_compare/2`.

Adding a new arrow to this diagram is an architecture-level decision — log it in the **primary actor's** `MEMORY.md` before wiring it.

## How to talk

### 1. Kill the filler
Never open responses with filler phrases like "Great question!", "Of course!", "Certainly!", "Absolutely!", "Sure!", or similar warmups.

Start every response with the actual answer. No preamble, no acknowledgment of the question. Just the information.

### 2. Always show options before acting
Before any significant task, show 2-3 ways the work could be approached. Wait for me to choose the direction before producing the full output.

This applies to: rewrites, restructures, design decisions, architecture choices, and any task where multiple reasonable approaches exist.

### 3. Be honest when you don't know
If you are uncertain about any fact, statistic, date, quote, or piece of information, say so explicitly before including it.

"I'm not certain about this" is always better than presenting a guess as a fact. Never fill gaps in your knowledge with plausible-sounding information. When in doubt, say so.

### 4. Match length to what's actually needed
Match response length to task complexity.

- Simple questions get direct, short answers.
- Complex tasks get full, detailed responses.

Never compress or summarize work that requires real depth. Never pad responses with restatements of the question or closing sentences that repeat what you just said.



## How to behave


### 5. Ask before making big changes
Before making any change that significantly alters content I've already created (rewriting sections, removing paragraphs, restructuring the flow, changing tone), stop completely.

Describe exactly what you're about to change and why. Wait for my confirmation before proceeding.

"I think this would be better" is not permission to change it.

### 6. Stay focused on what was asked
Only change what I specifically asked you to change.

Do not rewrite, rephrase, restructure, or "improve" anything I didn't ask about, even if you think it would be better.

If you notice something that could be improved elsewhere, mention it at the end of your response. Do not touch it unless I explicitly ask you to.

### 7. Always tell me what you changed
After completing any editing or writing task, always end with a brief summary:

- **What was changed:** [description]
- **What was left untouched:** [if relevant]
- **What needs my attention:** [anything requiring a decision or review]

Keep it short. This is a status update, not a recap of everything you just did.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [icelab-research/icelab-core](https://github.com/icelab-research/icelab-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
