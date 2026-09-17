---
trigger: always_on
description: OpenCutAgent (formerly EditAgent; the local folder is still `editagent/`) is a Premiere Pro CEP panel + Node MCP server that lets Claude edit a video on Premiere's **live timeline**: transcribe, mark retakes/silences, cut, and generate Remotion animations onto a track. The AI judgment is always **Claude itself** (this chat via the `ppro_*` tools, a headless `claude -p` spawned by the server, or the cloud proxy). There is no other LLM.
---

# Working in OpenCutAgent

OpenCutAgent (formerly EditAgent; the local folder is still `editagent/`) is a Premiere Pro CEP panel + Node MCP server that lets Claude edit a video on Premiere's **live timeline**: transcribe, mark retakes/silences, cut, and generate Remotion animations onto a track. The AI judgment is always **Claude itself** (this chat via the `ppro_*` tools, a headless `claude -p` spawned by the server, or the cloud proxy). There is no other LLM.

- **First time on this machine, or "make it work":** run the `/setup` skill (`.claude/skills/setup/`). It drives `install.sh` / `install.ps1 --check`, fixes the `[fix]` lines, verifies. The panel's header pulse icon (Health dropdown, `server/health.js` + the `health` RPC) shows the same prerequisites with green/red dots.
- `docs/ARCHITECTURE.md` explains how the pieces talk and why. Read it once.
- `docs/LESSONS.md` is the dated, full-detail history of every feature and failure (symptom, root cause, fix). **Search it before debugging anything that "used to work" or looks like a known symptom.** It is long on purpose; it is not loaded per session.
- **When you hit a new failure and find the fix:** add a one-line rule under the matching section below, and the full story to `docs/LESSONS.md` (and to the project memory file).

## Layout

```
Claude Code --stdio--> server/ (MCP + ws 127.0.0.1:3001 + ffmpeg/Scribe/claude -p) --ws--> cep-panel/client (UI) --evalScript--> cep-panel/host/premiere.jsx (ALL Premiere DOM access)
```

- `server/` - tools (`tools/`), RPCs the panel calls (`rpc/index.js`), review/segmentation (`review.js`, `transcription/`), silences (`silences.js`, `audio/`), headless AI (`ai.js`), cloud proxy client (`cloud.js`), animation agent (`animation/`), XML fast-apply (`rebuild.js`, `roundtrip.js`). State: `ctx.review`, `ctx.silence`, `ctx.panelOp` (edit lane), `ctx.animOp` (animation lane).
- `cep-panel/client/` - `main.js` + `index.html` + `styles.css` (token system, spec in `cep-panel/DESIGN.md`, gallery in `preview.html`). Debug handle in a browser: `window.__editagent = {Retake, Silence, AI, Anim, setConn}`.
- `animation-kit/` - Remotion template. At runtime it is synced to **`~/.opencutagent/animation-kit`** (the agent's cwd must be outside this repo or it auto-loads this file). Styles are drop-in packages `styles/<id>/{style.json,SKILL.md,src/}`; `n8n-brand`, `n8n-ui`, `n8n-game` are gitignored/local-only.
- `.claude/skills/` - the editing workflows; the same text is injected into headless calls (single source of truth).
- Backend (cloud mode, metered proxy, site): separate private repo `~/Documents/opencutagent_backend`.

## Server and connection (the #1 source of "why isn't it working")

The panel is ONLY a ws client. Nothing works until `server/index.js` listens on 3001. It is started by (1) the panel auto-start, (2) Claude Code via `.mcp.json` (needed for Sync mode; start Claude Code first), or (3) `npm start` / `./start-opencutagent.command`. One server per machine, operating on whatever sequence is active.

- Check who holds the port: `lsof -nP -iTCP:3001 -sTCP:LISTEN`. A relative `node server/index.js` command = started by hand in a terminal.
- **"panel not connected" though the panel's Health dot is green / "Unknown RPC method" after a code change:** a stale server (often from a SECOND Claude Code window) owns 3001 with old code. Keep ONE Claude Code window in the project, `pkill -9 -f "editagent/server/index.js"` once (does not match a hand-started server; kill that by PID), then `/mcp` reconnect `premiere`. Do not kill+probe repeatedly; every kill triggers a respawn.
- **"Waiting for server…"** = no server on 3001, not a code bug. Reopen the panel or start one by hand.
- A zombie MCP-spawned server that is alive but not listening (ppid = a `claude` process) is harmless.
- `.mcp.json` is gitignored (template `.mcp.json.example`); this Claude Code build does NOT expand `${VAR}`, use absolute paths.
- **Ground truth without the MCP:** kill the squatter and run a throwaway script importing `server/bridge.js` + `review.js` that binds 3001, waits for the panel to reconnect, dumps `getTimeline`/`buildReview`/`reconcile`, exits. Give it a dummy `rpcDispatcher`. Never "just connect and peek" at a running server: a second ws client hijacks the panel binding.
- **Live panel DevTools:** `cep-panel/.debug` + PlayerDebugMode expose the real panel at `http://localhost:8078` (`curl /json`, then `Runtime.evaluate` over its ws). Use this FIRST for "behaves weird only in Premiere"; it beats screenshot guessing.
- The panel-spawned server has stdout on /dev/null (no log) and inherits the bare GUI PATH; `server/paths.js augmentPath()` widens it. Reproduce PATH bugs with `env -i HOME=$HOME PATH=/usr/bin:/bin:/usr/sbin:/sbin node …`.

## Loading code changes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leonardogrig/opencutagent](https://github.com/leonardogrig/opencutagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
