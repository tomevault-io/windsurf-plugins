---
trigger: always_on
description: > Canonical repo-local AI context lives here. Keep tool-specific files like `CLAUDE.md` only as thin compatibility shims that point back to this file.
---

# AGENTS.md — RemoteLab Project Context

> Canonical repo-local AI context lives here. Keep tool-specific files like `CLAUDE.md` only as thin compatibility shims that point back to this file.

> **Read this file first.** It gives you everything you need to work on this project without exploring blindly.
> For deep-dive topics, reference docs are linked at the bottom.

---

## What Is RemoteLab

A web app that turns a real macOS/Linux machine into an AI automation workbench accessible from phone and desktop. It is designed first for people with repetitive digital work — including users who are not already AI power users — so they can bring a messy task, sample files, or a recurring chore and let the system help clarify the problem, shape an approach, and execute it on the machine.

**Not** a terminal emulator, a traditional editor-first IDE, or a chatbot. It's an **AI workbench / control console for human-AI collaboration** — the user does not need a perfect spec up front; RemoteLab should help discover the right problem, propose a workable workflow, and keep execution plus context coherent.

- Single owner, not multi-user
- First wedge: repetitive digital work that can be automated quickly
- Phone + desktop are both first-class control surfaces
- Node.js, no external frameworks (only `ws` for WebSocket)
- Vanilla JS frontend, no build tools

## Current Product Framing

- Default target user: someone with repeated digital chores, not necessarily an AI-native operator
- Core promise: a short conversation can eliminate hours of recurring manual work each week
- Interaction rule: AI should act like a product manager / solution designer, helping clarify the task instead of waiting for a perfect prompt
- Multi-session orchestration remains useful, but it is an enabling capability rather than the primary headline

## Current Product Development Loop

- Mainline work is continuous user discovery and feedback-driven iteration around mainstream guided automation on real machines.
- Prefer work that helps a normal user reach one trusted automation win over work that mainly showcases orchestration sophistication.
- Treat user feedback as a durable product asset; summarize reusable signals in repo-visible notes instead of leaving them only in private chat or machine-local memory.
- Keep raw identifying details out of shared notes; record sanitized product evidence and promote stable conclusions into canonical docs.
- See `notes/current/product-mainline.md` and `notes/current/user-feedback-log.md`.

## Documentation Rule

For setup, deployment, integration, and feature-activation docs, use a model-first, prompt-first shape:

- assume the operator is a human delegating to their own AI coding agent
- have the AI collect all required context in one early handoff whenever possible, instead of drip-feeding questions across many turns
- prefer one structured input packet from the human, then autonomous execution by the AI until completion or a true `[HUMAN]` checkpoint
- lead with a copyable prompt, one-round input requirements, target state, and explicit `[HUMAN]` checkpoints
- keep automatable command-by-command flow inside the AI conversation or scripts, not as a long manual cookbook
- minimize human interruption so the operator can hand off the task and come back only for approvals, browser-only actions, validation, or final handoff

---

## Architecture

```
Browser / app surface ──HTTPS──→ Cloudflare Tunnel ──→ chat-server.mjs (:7690)
                                                    │
                                      HTTP control plane + WS hints
                                                    │
                                      durable history + run state
                                            │
                                 detached runners normalize back to HTTP
```

### Chat Architecture

| Service | Port | Domain | Role |
|---------|------|--------|------|
| `chat-server.mjs` | **7690** | production chat domain | **Primary** — the shipped owner chat/control plane |

**Dev workflow**: use the normal `7690` service as the single chat/control plane. Restarting it should boot the current source tree directly; RemoteLab relies on clean restart recovery rather than a separate release snapshot or permanent validation plane.

**Self-hosting rule**: restarting the active chat server is acceptable when needed because runs reconcile back from durable state. Treat restart as a transport interruption with logical recovery, not as a reason to maintain a second permanent chat plane. Manual extra instances remain optional ad-hoc debugging tools only. See `notes/current/self-hosting-dev-restarts.md`.

---

## File Structure

```
remotelab/
├── chat-server.mjs          # PRIMARY entry point (HTTP server, port 7690)
├── cli.js                   # CLI entry: `remotelab start|stop|restart|setup|...`
├── generate-token.mjs       # Generate 256-bit access tokens
├── set-password.mjs         # Set password-based auth
│
├── chat/                    # ── Chat service modules ──
│   ├── router.mjs           # All HTTP routes & API endpoints (538 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ninglo/remotelab](https://github.com/Ninglo/remotelab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
