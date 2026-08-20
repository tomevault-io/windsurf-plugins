---
trigger: always_on
description: Guidance for Claude Code working in the BlitzOS repo.
---

# CLAUDE.md — BlitzOS

Guidance for Claude Code working in the BlitzOS repo.

## What this is (V1 = island-only)

BlitzOS is an Electron macOS **dynamic island** (the notch) for human+agent collaboration. The human drives it, and so can an AI agent over the [agent-socket](https://agentsocket.dev) relay (paste a URL / "connect to blitz os", no MCP) or the localhost control server. **The island IS the whole UI** — there is NO canvas, no infinite plane, no surfaces-on-a-canvas, no pan/zoom, no slot lattice, no stages, no workspace switcher, no dock/Sidebar, no Overview, no radial menu, no folders/annotations. (All of that was the pre-V1 desktop; it was cut. See "what got cut" under Gotchas.)

**The island.** A black pill at the notch that opens — on hover, or ⌥Space — into a body-portal chassis: hover shows a **home grid of widget icons** (V1 ships ONE functional icon, **Chat**, flanked by dotted placeholders for the agent-made widgets that come post-V1); ⌥Space just shows/hides the island (restoring the last view+tab), never spawning or mutating state. Chat is the agent session UI: a tab strip (a pen button that spawns a brand-new agent and enters it, plus one tab per live agent — Blitz '0' first), an iMessage-style transcript interleaved with the narrator's plain milestone steps, a live status line, a steer bar, a "Details" expand for raw tool rows, and a "+" attach panel for **connections** (browser/computer use). Web surfaces are GONE — the agent works the user's REAL browser (the connector extension) and native apps (the computer-use helper) instead. **Widgets are DEFERRED** (experimental, post-V1): the agent generating/pinning its own island widgets and apps is not in V1, which ships only Chat.

**Agent-runtime model (why BlitzOS exists).** BlitzOS is an *OS for an agent*: it turns ANY connected agent (Claude Code today over agent-socket, any agent, a built-in chat client later) into an autonomous one with **zero per-task code**. The agent supplies intelligence; BlitzOS supplies the loop. Four parts: **syscalls** (the agent's tools/hands — chat, connections, terminals, agents, workflows), **perception** (a content-agnostic world stream, the agent's eyes), **a scheduler** (coalesced "moments" that *wake* the agent with a snapshot, the interrupt), and the **agent as swappable policy** (it decides significance and action). The whole point is **out-of-distribution generalization**: perception is dumb-but-rich and the agent decides what matters, so a new task (coach my chess, draft this email, summarize this PDF) needs no new BlitzOS code. **Never hand-build a per-task watch loop** — make perception and wake general, and let the agent's policy handle the task. See "Agent runtime" under Architecture.

Prototype stage (version 0.0.1); the island/runtime agent doctrine lives in `src/main/blitzos-agents.md` (match its island framing).

## Commands

```bash
npm run dev        # electron-vite dev (the GUI; macOS only) — predev ensures the native helpers
npm run build      # electron-vite build → main + preload + renderer to out/
npm run typecheck  # tsc --noEmit -p tsconfig.json
npm run check      # typecheck + parity (scripts/check-parity.mjs) + build — the green-it gate
npm run dist       # package BlitzOS.app (signed+notarized when ~/.zshrc Apple creds present) -> release/*.zip
```

There is **no display in CI / headless sandboxes**, so you cannot see the GUI. Verify behavior instead by: launching `npm run dev > /tmp/aos.log 2>&1`, then reading the log for `did-finish-load`, the printed control-API token, and the agent-socket paste URL; and by driving the app via the control API or agent-socket tools (below) and checking `list_state`. Never claim the pixels look right, that's the user's to confirm.

## Stack & layout

electron-vite + React + TypeScript + zustand.

```
src/main/        Electron main (Node)
  index.ts         the one BrowserWindow, notch overlay, boot-task seam, wires everything
  notch-overlay.ts the dynamic-island overlay window + the always-interactive notch hit-window
  osActions.ts     control plane: IPC mutations + getState, the live-WebContents registry
  control-server.ts  localhost HTTP control API (trusted local-agent path; mints session.json)
  agentSocket.ts   connects to the agent-socket relay; tools -> osActions (remote agent path)
  os-tools.mjs     THE one shared agent-tool registry (both transports — see Architecture)
  agent-runtime.mjs / terminal-manager.mjs   the managed-agent backends + tmux terminals
  events.ts / perception-core.mjs   the perception kernel → moments → /events wake
  blitzos-agents.md   the runtime agent's island doctrine (what an agent reads on connect)
src/preload/index.ts   contextBridge api: onAction, sendState, onAgentSocketUrl, notch.*
src/renderer/src/
  App.tsx          the notch wiring + body-portal of the island; NO canvas render
  notch/           THE island UI (KEPT): NotchHost / IslandHome / IslandPanel / ChatInput / AttachPanel
  store.ts         zustand — runtime panels (chat/terminal/inbox) + viewport (mid-migration, see Gotchas)
  components/{ConnectPicker,Icons}.tsx
vendor/agent-socket-sdk/   vendored @agent-socket/sdk dist (see "Gotchas")
```

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blitzdotdev/BlitzOS-oss](https://github.com/blitzdotdev/BlitzOS-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
