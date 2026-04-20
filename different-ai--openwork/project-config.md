---
trigger: always_on
description: OpenWork helps users run agents, skills, and MCP. It is an open-source alternative to Claude Cowork/Codex as a desktop app.
---

# AGENTS.md

OpenWork helps users run agents, skills, and MCP. It is an open-source alternative to Claude Cowork/Codex as a desktop app.

## What OpenWork Is

OpenWork is a practical control surface for agentic work:

* Run local and remote agent workflows from one place.
* Use OpenCode capabilities directly through OpenWork.
* Compose desktop app, server, and messaging connectors without lock-in.
* Treat the OpenWork app as a client of the OpenWork server API surface.
* Connect to hosted workers through a simple user flow: `Add a worker` -> `Connect remote`.

## Core Philosophy

* **Local-first, cloud-ready**: OpenWork runs on your machine in one click and can connect to cloud workflows when needed.
* **Server-consumption first**: the app should consume OpenWork server surfaces (self-hosted or hosted), not invent parallel behavior.
* **Composable**: use the desktop app, WhatsApp/Slack/Telegram connectors, or server mode based on the task.
* **Ejectable**: OpenWork is powered by OpenCode, so anything OpenCode can do is available in OpenWork, even before a dedicated UI exists.
* **Sharing is caring**: start solo, then share quickly; one CLI or desktop command can spin up an instantly shareable instance.

## Core Runtime Model (Updated)

OpenWork now has three production-grade ways to run the same product surface:

1. **Desktop-hosted app/server**
   - OpenWork app runs locally and can host server functionality on-device.
2. **CLI-hosted server (openwork-orchestrator)**
   - OpenWork server surfaces can be provided by the orchestrator/CLI on a trusted machine.
3. **Hosted OpenWork Cloud server**
   - OpenWork-hosted infrastructure provisions workers and exposes the same remote-connect semantics.

User mental model:

* The app is the UI and control layer.
* The server is the execution/control API layer.
* A worker is a remote runtime destination.
* Connecting to a worker happens through `Add worker` -> `Connect remote` using URL + token (or deep link).

Read `ARCHITECTURE.md` for runtime flow, server-vs-shell ownership, and architecture behavior. Read `INFRASTRUCTURE.md` for deployment and control-plane details.

## Why OpenWork Exists

**Cowork is closed-source and locked to Claude Max.** We need an open alternative.
**Mobile-first matters.** People want to run tasks from their phones, including via messaging surfaces like WhatsApp and Telegram through OpenCode Router.
**Slick UI is non-negotiable.** The experience must feel premium, not utilitarian.

## Agent Guidelines for development

* **Purpose-first UI**: prioritize clarity, safety, and approachability for non-technical users.
* **Parity with OpenCode**: anything the UI can do must map cleanly to OpenCode tools.
* **Prefer OpenCode primitives**: represent concepts using OpenCode's native surfaces first (folders/projects, `.opencode`, `opencode.json`, skills, plugins) before introducing new abstractions.
* **Web parity**: anything that mutates `.opencode/` should be expressible via the OpenWork server API; Tauri-only filesystem calls are a fallback for host mode, not a separate capability set.
* **Self-referential**: maintain a gitignored mirror of OpenCode at `vendor/opencode` for inspection.
* **Self-building**: prefer prompts, skills, and composable primitives over bespoke logic.
* **Open source**: keep the repo portable; no secrets committed.
* **Slick and fluid**: 60fps animations, micro-interactions, premium feel.
* **Mobile-native**: touch targets, gestures, and layouts optimized for small screens.

## Task Intake (Required)

Before making changes, explicitly confirm the target repository in your first task update.

Required format:

1. `Target repo: <path>` (for example: `_repos/openwork`)
2. `Out of scope repos: <list>` (for example: `_repos/opencode`)
3. `Planned output: <what will be changed/tested>`

If the user request references multiple repos and the intended edit location is ambiguous, stop after discovery and ask for a single repo target before editing files.

## New Feature Workflow (Required)

When the user asks to create a new feature, follow this exact procedure:

1. Make sure you are up to date on all submodules and repos synced to the head of remotes.
2. Create a worktree.
3. Implement the feature.
4. Start the OpenWork dev stack via Docker (from the OpenWork repo root): `packaging/docker/dev-up.sh`.
5. Use Chrome MCP to fully test the feature: `.opencode/skills/openwork-docker-chrome-mcp/SKILL.md`.
6. Take screenshots and put them in the repo.
7. Refer to these screenshots in the PR (only if relevant in the UI).
8. Always test the flow you just implemented.

If you cannot complete steps 4-8 (Docker, Chrome MCP, missing credentials, or environment limitations), you must say so explicitly and include:

* which steps you could not run and why
* what you verified instead (tests, logs, manual checks)
* the exact commands/steps the user should run to complete the end-to-end gate

## Pull Request Expectations (Fast Merge)

If you open a PR, you must run tests and report what you ran (commands + result).

To maximize merge speed, include evidence of the end-to-end flow:

* Ideally: attach a short video/screen recording showing the flow running successfully.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [different-ai/openwork](https://github.com/different-ai/openwork) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
