---
trigger: always_on
description: Bun backend: HTTP API + WebSocket fan-out over herdr's unix-socket API, plus the PTY attach sidecar. The attachment lifecycle and transport quirks live here and nowhere else.
---

# server/

Bun backend: HTTP API + WebSocket fan-out over herdr's unix-socket API, plus the PTY attach sidecar. The attachment lifecycle and transport quirks live here and nowhere else.

## ANTI-PATTERNS
- Never let an attachment outlive its last client. Cleanup paths: `detach`/`close` for members (`detach()` closes on an empty client set), `releaseUnclaimed` for an attach cancelled mid-creation, `onExit` for a pty that died, `stop()` for everything.
- Never `attachments.set` outside `spawnAttachment`, and never await between that `set` and `new PtySession(...)`: the record's `pty` is a placeholder until then.
- Never add the client to `attachment.clients` before checking `client.data.attached` still holds the pane after the await.
- Always resize the pty from the geometry of an INTERACT client's attach (`attach` handler calls `resizePty` when `mode === "interact"`); an observe attach never resizes. Never spawn `herdr terminal attach` without cols/rows.
- Answer in-band: handler failures are sent as `{type:"error"}` frames. The only deliberate closes are 4008 (output stall/overflow), 1008 (device revoked) and 1012 (relay/PC gone).
- Keep `herdr/` free of HTTP concepts — it speaks only herdr's socket protocol; HTTP shaping belongs in index.ts.

## CONVENTIONS
- `attachments: Map<paneId, PaneAttachment>` holds `{ pty, clients, cols, rows, replay }`. `cols`/`rows` are the pty's current grid: `resizePty` updates them and broadcasts `pane-geometry` to the pane's clients whenever they change. `replay` is a bounded 256KB tail sent to clients that attach late; `pty-exit` broadcasts first, then closes the attachment.
- `createServer` starts `startStatusCollector` (collector.ts): per-pane `agent_status_changed` for EVERY pane in the snapshot plus a global pane.created/pane.closed/pane.exited subscription. Every stream reopens itself when herdr closes it (events_lost, restart) with 250 ms→5 s backoff; a lifecycle stream that comes back reconciles, and a lost status stream resyncs from a snapshot taken after the new subscription started. Its callbacks `broadcastAll` `pane-status` / `pane-exited` / `session-changed` to every connected client — `pane-status` no longer depends on attachment. The client set is tracked in a `clients` Set (open/close).
- Sidecar protocol (pty-host.mjs): argv `<cols> <rows> <command> [args...]`; stdin newline-JSON frames — `{"t":"i","d":"<text>"}` writes, `{"t":"r","c":cols,"r":rows}` resizes; stdout is raw pty bytes unmodified; exit mirrors the child's code; sidecar clamps cols/rows to 1..1000.
- Output control adds `{"t":"p","paused":true|false}`. Parent pause and stdout backpressure are independent; both must clear to resume. Kill resumes the reader to observe EOF and has a bounded forced cleanup. PtySession awaits stdout EOF before reporting process exit.
- ACK-capable attaches carry `flow_control:"ack"`; `pty-data.flow` identifies the subscription and cumulative UTF-8 byte offset. `pty-ack` releases only already-sent credit in that subscription. Pause at 256 KiB, resume at <=64 KiB, close consumers blocked for 2s with code 4008; 1 MiB hard application/transport budgets. `-1` from Bun send is already queued, never retransmit it. Legacy clients have the transport cap only. See docs/terminal-flow-control.md and server/output.contract.test.ts.
- `HerdrError.code` → HTTP status (http.ts `errorResponse`): `connect_failed`/`timeout` → 502; other HerdrError codes → 404; anything else → 500 `internal_error`.
- Auth: auth.ts `isAuthenticated` accepts the `herdr_web_token` cookie or an Authorization Bearer (length-safe compare); access.ts `decideAccess` (token → device cookie → Tailscale owner → loopback local → pairing) runs for /ws and every /api/* route except /api/health, /api/auth and /api/devices/pair; static files stay open.
- An unauthenticated /ws upgrade is refused with a plain-text 401 `unauthorized` (the WS client never parses a body); every other refusal is the JSON envelope.
- `terminalInfoFor` resolves pane_id → `{ terminalId, rect }` from one fresh `session.snapshot` (rect = the pane's `layouts[].panes[].rect`, null when absent); missing pane → `pane_not_found`, pane without terminal → `no_terminal`.
- The layout `rect` is NOT the terminal's current size (measured on herdr 0.9.3): it stayed 120x40 while a `terminal attach` sized the pane 100x30, then 140x40, and the pane kept 140x40 after that attach left. Only the attached client's size is real; nothing in the API tells another bridge's size. An attachment started again (held retry, live handoff) therefore reuses its own cached `cols`/`rows` (#363 review).
- Attachment creation: `ensureAttachment` returns the existing record, else joins the in-flight creation in `pendingAttachments` (cleared in `finally`), else starts `spawnAttachment`. An observe-first creation spawns at the pane's rect (fallback 80x24), never the observer's grid; an interact-first one at the client's grid.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devswha/herdr-web-ui](https://github.com/devswha/herdr-web-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
