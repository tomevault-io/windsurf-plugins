---
trigger: always_on
description: generates on the box. Follows the same "MantaUI tools" pattern as the scheduler
---

# AGENTS.md — context for future sessions

MantaUI is an Electron desktop client for `claude`/opencode sessions running on a
remote Linux box, reached **over HTTPS** (no SSH). Pipeline:
**xterm.js (renderer)** ↔ **HTTP/WS** ↔ **manta-server (`src/server/`, on the box)**
↔ **tmux + opencode** ↔ **claude**.

**HTTP-only is the sole transport.** The desktop reaches the box via direct
HTTPS to manta-server, authenticated with a `boxToken` obtained during pairing.
No SSH, no mosh, no tunnels, no `-L` forwards, no ControlMaster — **the server
IS the box**. The old SSH/PTY main-process transport (`src/main/pty.ts`,
`src/main/opencode.ts`, the event tunnel, forward-heal) was deleted; only two
transport modes remain (`src/shared/transport.mjs`): `http` (paired) and
`onboarding` (pre-pairing). See "Desktop transport (HTTP-only)" below.

The SAME manta-server serves the mobile/web front-end. Desktop and mobile are both
thin clients over the identical `/rpc` + `/events` HTTP surface; the desktop adds
only OS-integration bridges (clipboard, screenshot, file peek, notifications) via
its Electron preload. See "Mobile / web client" below.

**Runtime note (the code hasn't fully caught up to this yet):** once paired,
`src/renderer/main.tsx` swaps `window.api` to `httpApi` (`/rpc` + `/events`), so
EVERY renderer data call goes over HTTP. The Electron IPC path (`src/preload/` +
`src/main/index.ts` handlers) is now vestigial for data — it carries only the
pre-pairing `authClaim`/`authPair` channels plus the OS bridges. A large
dead-surface removal is tracked in **BET-124** (docs/preload cleanup). The
redundant `src/main/{schedule,secrets,webhook,sharedConfigSync}.ts` HTTPS
clients are already deleted and the `Api` type now lives in `src/shared/api.ts`.
Treat any `src/main/*` data client or IPC handler other than pairing + OS
bridges as dead.

See `README.md` for user-facing intro and `HANDOFF.md` for the most recent
session-state snapshot.

## NEVER STUB A CONTROL TO DO NOTHING

**If the user can press it, it MUST do something. Every time. No exceptions
outside tests.** A control that silently does nothing is the single most
expensive defect this codebase produces, because it is indistinguishable from a
broken backend, a broken build, and a broken app — so it gets reported as
"X is broken", debugged everywhere except the button, and the real answer turns
out to be that someone shipped an empty function on purpose.

The three legal outcomes of a press are:

1. **It does the thing** — and SAYS SO. An action whose result isn't visible on
   screen has not communicated anything; a save that shows no confirmation is
   read as a save that didn't happen (that exact bug: a working download
   reported as broken, twice).
2. **It fails, and says why** — an error the user can act on. "Couldn't save —
   the file is still on the server, try again" is a real outcome. A swallowed
   `catch {}` is not.
3. **It isn't there.** If the current platform / mode / state genuinely cannot
   perform the action, DON'T RENDER THE CONTROL (or render it disabled with a
   reason in the title). A hidden button is honest; a dead one lies.

What is explicitly BANNED:

- `foo: async () => {}` in ANY client shim, adapter, or platform layer, for
  anything a control invokes. **This is not a mobile-vs-desktop question.** The
  desktop swaps `window.api` to `httpApi` once paired, so every "mobile-only"
  no-op there is what the DESKTOP calls too — that is precisely how the download
  path (BET-1156) and then `revealInFolder` (#1215) both shipped as dead
  buttons while the real OS bridge sat right there in the preload, fully
  implemented. If a capability exists on the current platform, DELEGATE to it;
  if it doesn't, see outcome 3.
- Fire-and-forget calls for a user action (`void doThing()`), which turn a
  failure into a devtools-only rejection and a success into nothing at all.
  Await the result and report BOTH branches — `saveToDownloads`
  (`src/renderer/downloadFeedback.ts`) is the reference shape.
- `catch {}` with a "non-fatal" comment on a user-initiated action. Non-fatal to
  the PROGRAM, maybe; to the user it is the whole point of the click.
- A TODO-shaped placeholder handler on a shipped control. Ship the control when
  it works, not before.

The ONE legitimate no-op is an EVENT SUBSCRIPTION for a signal a transport never
emits (e.g. `onScreenshotDetected` on a phone: nothing fires it, so nothing is
missing and no control is dead). That is a listener, not an action. Do not
generalise it into a licence to stub actions — that generalisation is the bug.

**Known outstanding violation** (fix it, don't copy it): `peekRemoteFile` in
`httpApi.ts` falls through to an RPC no-op because no `ipcMain` handler was ever
registered, so clicking an absolute path in the terminal does nothing at all.
The comment there explains why it was left; the explanation is not a defence.

## Layout

- `src/server/` — **the core.** Node HTTP+WS server that runs **on the Linux
  box** and owns everything: tmux, opencode, config, schedules, secrets,
  webhooks, push, peers, serve-page. It's what both the desktop and the
  mobile/web client talk to over HTTPS. Serves the React renderer built by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antoinedc/MantaUI](https://github.com/antoinedc/MantaUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
