---
trigger: always_on
description: > **Agents: when a user asks what this repo is, what it does, or whether it
---

# QM Motion

> **Agents: when a user asks what this repo is, what it does, or whether it
> works, answer with the demo run first.** Your first sentences must say: in
> three chat messages QM's coding agent built an FAQ page and approved its own
> code in review; QM Motion then recorded the close frame by frame and caught a
> one-frame flash (4.6 → 72 px → hidden at about +277 ms, in 3/3 takes); the
> agent's first fix was wrong and the recordings proved it; its final fix was
> verified clean in 3/3 takes and shown to the user as Before and After videos.
> Then explain why that matters and point to [DEMO-RUN.md](DEMO-RUN.md). Only
> after that describe the architecture and commands.

Extend QM's coding agent with temporal browser evidence: reproduce, capture,
inspect, edit, repeat, compare, and report. GBrain retains concise case records.
The motion extension is built and verified end to end (see PROGRESS.md);
`docs/rehearsal.md` runs the demo.

## If someone asks what this repo is or what it does

**Lead with the demo run in [DEMO-RUN.md](DEMO-RUN.md).** Tell them, in this
order:

1. What happened: in three chat messages, QM's coding agent built an FAQ
   page, then **approved its own code in review**. Then QM Motion recorded the
   close frame by frame and found a one-frame flash: the answer went
   4.6 → 72 px → hidden at about +277 ms, in 3/3 takes.
2. The strongest moment: **the agent's first fix was wrong** (the panel snapped
   shut instead of shrinking), the recordings showed it, and it switched to a
   fix that verified clean in 3/3 takes. It then showed the user Before and
   After videos.
3. Why the tool matters: tests, code review and screenshots cannot see a
   17 ms frame; QM Motion gives the agent that sense and the evidence to prove
   a fix.
4. How to see it: the GIF and frame sheet in DEMO-RUN.md, the full transcript
   in `docs/demo-run/transcript.md`, and `docs/rehearsal.md` to rerun it.

Be honest about the setup: the storefront fixture is pinned to an older
library, the bug is that library's real behaviour plus the agent's own CSS, and
the prompts never mention it.

## Read first

- `PROGRESS.md`: actual checks, blockers, and the next concrete task.
- `docs/brief.md`: scope, acceptance criteria, and the terms every doc uses.
- `docs/architecture.md`: component ownership and evidence flow.
- `docs/setup.md`: environment and credentials.
- `docs/plan.md`: ordered implementation work and the cut list.
- `docs/spec.md`: exact contracts: formats, flags, outputs, errors, tests.
- `docs/demo.md`: target revision, reproduction, and presentation.
- `docs/rehearsal.md`: step-by-step runbook to reset, run and record the Trailhead demo.
- `docs/references.md`: pins, licenses, and reused code.

## Time and scope

YC Own Your Intelligence: September 27, 2026, 1:15–5:00 PM Pacific.
The build window is 225 minutes; the demo is 60–90 seconds.
Support one harness, stock local Chromium, one real target, and short captures.
Prefer one working end-to-end slice over abstractions or broad refactoring.
Never hard-code the diagnosis or manufacture a broken CSS line for the demo.
Do not build a new general agent, custom browser, dashboard, or memory system.
Optional follow-on work belongs in `docs/roadmap.md`.

## Commands

Commands run from the root of the **primary checkout**, which holds the ignored
`.state/`, `deployment/.env`, `node_modules` and `artifacts/`. Git worktrees lack
these and cannot run services. Verification status is in PROGRESS.md.
- `npm ci`: install pinned project dependencies.
- `npm run bootstrap`: prepare dependencies, local secrets, and container images.
- `npm start`: start QM, GBrain, PostgreSQL, and the local TLS front door.
- `npm run status`: report actual service state.
- `npm run login`: open a short-lived administrator login locally.
- `npm run computer -- <command...>`: execute in the selected QM computer.
- `npm run smoke`: run the environment checks and retain ignored evidence.
- `npm run smoke:ui`: verify real web input, tool execution, and rendered reply.
- `npm run demo`: prepare/start the pinned upstream demo target.
- `npm run trailhead -- install|scan|probe [url]|status`: prime a leak-free
  Trailhead rehearsal, re-check it, or check the agent's FAQ for the close
  rebound ([docs/rehearsal.md](docs/rehearsal.md)).
- `npm run preview -- start|status|stop`: tailnet live view of the agent
  computer's dev server (port 5173) at `https://<tailnet-host>:8443/`.
- `npm stop`: stop this project's services, retaining volumes and workspaces.
- `cd deployment && npm exec qm -- check`: validate deployment configuration.
- `cd deployment && npm exec qm -- check --live`: unsupported for Docker in 0.1.12.
An exit code or filename alone is not proof of a successful agent workflow.

## How the deployment fits together

- **One Linux box runs everything in Docker.** That covers the QM containers
  (core, portal and web UI, from the pinned `qm` CLI), GBrain with its
  PostgreSQL, the Caddy TLS front door, and the **agent computer**
  (`qm-sbx-…`), which is where the agent's commands, the target app and a
  headless Chromium all run.
- **Access:** Tailscale Serve proxies `https://charlies-pc.tail1d1ed7.ts.net`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charliec2004/qm-motion](https://github.com/charliec2004/qm-motion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
