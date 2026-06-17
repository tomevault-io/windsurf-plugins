---
trigger: always_on
description: handles everything that requires judgment or creativity.
---

# boxBot — Project Principles

## What Is boxBot?
An open-source Claude agent that lives in an elegant wooden box, built on a
Raspberry Pi 5 with the Claude Agent SDK. It sees, hears, remembers, and
communicates — acting as an ambient household assistant that recognizes the
people around it and proactively helps them.

## Core Design Principles

### 1. Security First
All communication with the outside world flows through **structured,
authenticated paths only**:
- **Voice** — direct conversation via microphone and speaker
- **Camera** — passive perception, never streamed or shared externally
- **Button inputs** — physical interaction on the box
- **WhatsApp** — registered users only; unknown numbers are hard blocked
  (silent drop, no response, no information leakage)

There is **no open network listener, no web UI, no SSH-by-default**. The
only open port is the WhatsApp webhook (required by the API).

**User registration** uses single-use, time-limited codes:
- First admin: code displayed on screen during setup (physical presence)
- New users: admin generates code, shares it out-of-band, new user texts
  it to BB. BB never initiates contact with unknown numbers
- See [docs/user-registration.md](docs/user-registration.md) for details

**WhatsApp is a privileged channel, not a proxy.** Email, calendar, RSS,
and other "inbox" services are **skills** running in the sandbox with their
own credentials. BB may relay results via WhatsApp, but data fetching never
touches the messaging path.

### 2. Slim Core, Flexible Extensions
The repository stays focused on the core agent loop, hardware abstraction, and
plugin interfaces. Features are added through three extension systems:
- **Tools** — always-loaded core capabilities (switch display, run script,
  send message, manage memory/tasks)
- **Skills** — modular, optional capabilities the agent can invoke (weather
  lookup, reminders, home control, etc.)
- **Displays** — swappable screen layouts the agent rotates through or selects
  contextually

Tools are the agent's hands — always attached, always available. Skills are
items it picks up — modular, swappable, contributed by the community. Both
use simple base classes. Skills and displays use auto-discovery so
contributors can add them without touching core code.

### 3. Tools, SDK, and Skills

Three layers, each with a different loading strategy and context cost.
The big shift (2026-04): most capability lives in the `bb` Python
package (sandbox SDK), reached through `execute_script`. Only genuinely
hot-path or security-sensitive operations remain as standalone tools.

**Tools** (`src/boxbot/tools/`) — 7 always-loaded tools:
- `execute_script` — primary gateway: run Python in the sandbox with
  the `bb` package (photos, camera, workspace, display, memory, tasks,
  skill, secrets, packages, integrations). Composes many ops per turn.
- `switch_display` — change what's on the 7" screen (hot-path singleton)
- `identify_person` — register / correct a speaker's identity; returns
  a cropped still of the speaker on new-person outcomes so the agent
  can note appearance into memory
- `manage_tasks` — triggers + to-dos (hot-path, every-turn)
- `search_memory` — memory lookup (hot-path, every-turn)
- `web_search` — web fetch + small-model content firewall
- `load_skill` — progressive-disclosure entry point (loads a skill's
  SKILL.md and optional sub-files into context on demand)

Speech and WhatsApp replies flow through the structured output
(`response_text` / `outputs` array) rather than tools — see
`output_dispatcher` and the `tools/registry.py` module docstring.

**SDK** (`src/boxbot/sdk/`, importable as `boxbot_sdk` or just `bb`
inside `execute_script`) — the constrained, immutable Python API:
- `bb.workspace` — filesystem-backed notebook (notes, CSVs, saved
  images). Path-safe, quota-capped, grep-searchable. The "now look it
  up" counterpart to memory's "rings a bell."
- `bb.camera` — `capture` / `capture_cropped` stills; images attach
  straight to the tool result so the agent sees pixels
- `bb.photos` — search, get, view (attaches pixels), show_on_screen
- `bb.display` — declarative block-based display builder + preview
- `bb.memory` — save/search/invalidate (shares backend with tool)
- `bb.tasks` — triggers + to-dos (shares backend with tool)
- `bb.skill` — create new skills at runtime
- `bb.integrations` — list / call / create / update / delete data-pipe
  integrations (sandbox-runnable manifest+script bundles); read
  execution logs for self-debugging. **Calendar lives here**:
  `bb.integrations.get("calendar", action="list_upcoming_events", …)`.
- `bb.secrets` — write-only credential storage
- `bb.packages` — request package install (human approval required)

The SDK communicates with the main process through structured JSON on
stdout + stdin (streaming, bidirectional). `execute_script` reads
action lines line-by-line, dispatches to per-module handlers, writes
JSON replies back to the sandbox, and collects image attachments into
a multimodal tool result. The agent never writes code that runs in the
main process.

**Self-documentation:** the `skills/bb/` skill is the agent's map to
the `bb` package. Its `SKILL.md` is injected into the system prompt at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dv-hart/boxBot](https://github.com/dv-hart/boxBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
