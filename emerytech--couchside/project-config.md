---
trigger: always_on
description: > **This file governs how you work, not what you build.**
---

# Couchside — Claude Instructions

> **This file governs how you work, not what you build.**
> Read it fully at the start of every session before touching any code.

Couchside turns a phone into the dashboard, remote console, and game controller for a
living-room Linux box (SteamOS / Bazzite / Steam Deck / any systemd machine). A native
iOS + Android app talks to a dependency-free Python service on the box. LAN-only,
token-authed, no cloud, no accounts, no analytics.

---

## ⛔ CRITICAL CONSTRAINT — READ BEFORE ANYTHING ELSE

**The agent executes only what is on an explicit allowlist. Nothing a client sends ever
becomes a command, a path, or a shell string.**

The agent runs as the user's desktop account on their gaming machine. It can launch
programs, switch sessions, reboot, and power off. It sits on a home LAN behind one bearer
token. If a client can steer it to run something arbitrary, an attacker on that LAN owns
the box — and the blast radius is somebody's personal computer, not a sandbox. There is no
cloud tier to revoke, no account to lock: a shipped hole stays open until the user updates
the agent themselves.

**Treat every violation as a zero-tolerance bug.** If you are ever unsure whether code
respects this, STOP and ask before writing or executing it.

---

## 1. Startup Sequence (run at the start of EVERY session)

1. **Read all memory files** in `docs/memory/`. Create missing ones with sensible defaults.
2. **Read `docs/ROADMAP.md`** — planned, in progress, done.
3. **Read `docs/BUILD_LOG.md`** — what was last worked on, where things were left.
4. **Scan the source area** you're about to touch — read existing files before creating new ones.
5. **Identify the allowlist pattern** already used in that area (launcher ids, action ids,
   menu ids, TV ops — each has one).
6. **State your plan** in one short paragraph before writing code: what, which files, how it
   fits existing patterns, how the allowlist constraint is maintained. Wait for confirmation
   if anything is architecturally ambiguous.

Note: `app/CLAUDE.md` carries an app-specific instruction (read the versioned Expo SDK 57
docs before writing app code). It is not superseded by this file.

## 2. Memory Files

| Path | Contents |
|------|----------|
| `docs/memory/ARCHITECTURE.md` | Stack, directory layout, key patterns, integrations |
| `docs/memory/CONVENTIONS.md` | Coding conventions, naming, test + PR + release rules |
| `docs/memory/DEPENDENCIES.md` | Every meaningful dependency, version, why |
| `docs/memory/DECISIONS.md` | **LOCAL — gitignored.** Append-only log of significant decisions |
| `docs/memory/KNOWN_ISSUES.md` | **LOCAL — gitignored.** Numbered KI-### registry: bugs, limitations, debt |

**Three files are deliberately untracked** (`docs/memory/KNOWN_ISSUES.md`,
`docs/memory/DECISIONS.md`, `docs/BUILD_LOG.md`). This repo is public, and a numbered
registry of where a LAN-exposed daemon is soft is not something to publish — the agent runs
as the user's desktop account and can reboot their machine. They exist on the maintainer's
machine and are read at the start of every session. **If you are in a fresh clone they will
be absent: create them rather than assuming there is no history**, and do not "helpfully"
commit them.

Update the relevant file any time you add a dependency, establish a pattern, make a decision,
or find an issue. **Never let these go stale.**

## 3. Allowlist Rules — never violate

1. **A client-supplied identifier is looked up, never interpolated.** Ids index a frozen
   set or dict defined in the agent source (`_STEAM_MENU_IDS`, `ACTIONS`, `LAUNCHERS`,
   the TV op tables). An id that is not present is a 404 — never a pass-through.
2. **`subprocess` is called with an argv LIST. Never `shell=True`, never a formatted
   command string.** The binary and its arguments are chosen by the agent; user input
   only ever selects *which allowlisted entry* runs.
3. **Never widen an allowlist to a pattern.** No globs, no prefix matches, no "anything
   under this namespace". Adding a capability means adding an explicit entry.
4. **Every route that changes state requires the bearer token.** `/api/ping` is the only
   deliberate pre-auth endpoint. `/pair` is loopback-only AND checks the Host header
   (anti-DNS-rebinding) because it renders the token.
5. **Paths derived from client input are contained.** Resolve, then verify the result is
   still inside the intended root before opening it.
6. **Reject rather than sanitise.** If input is not exactly a known-good value, return an
   error. Do not attempt to clean it up.
7. **Degrade closed.** When a probe fails or `/proc` is unreadable, return "unavailable",
   never "allowed".
8. **If you find existing code violating these rules**, do not replicate it. Add a KI-###
   entry with `Impact: high` and flag it immediately.

## 4. Hard Constraints — Never Violate

- **The gamepad / trackpad input path is safety-critical.** It is the most stateful,
  most concurrent code in the project and the source of most escaped bugs: half-dead
  sockets, promote/demote races, leaked uinput devices. Changes here require tests for
  the lifecycle (create → hold → hand off → reap), not just the happy path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emerytech/couchside](https://github.com/emerytech/couchside) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
