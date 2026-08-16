---
trigger: always_on
description: Unpeel is an AI-native, terminal-first workspace for running CLI agents. It is not tied to any single audience — developers, creatives, and non-coders alike find a home in it — and the agents inside it are for any task, not just code.
---

# AGENTS.md

Unpeel is an AI-native, terminal-first workspace for running CLI agents. It is not tied to any single audience — developers, creatives, and non-coders alike find a home in it — and the agents inside it are for any task, not just code.

This file documents the current agent/session system in Unpeel. It is meant to be a practical map of what exists today, where it lives, and what has to stay aligned when you change it.

## Product Philosophy

Unpeel is like Codex or Claude, but **for everything** — not just code. The
terminal-hosted agent is the product surface for any task a CLI agent can do
(research, writing, ops, data, automation), not a coding IDE.

Concretely, this means Unpeel will **never** grow code-editor affordances:

- no diff viewers
- no file/folder tree or file browser
- no source-code editor panes, language tooling, or symbol navigation
- no other "IDE" chrome that frames Unpeel as a code tool

The agent talks to its tools through the terminal; Unpeel's job is to make that
agent accessible and productive for everyone — regardless of domain. When
proposing UI, do not add code-centric views. If a feature only makes sense for
coding, it does not belong in Unpeel.

> **Architecture note (2026-06-13):** the original Tauri + Svelte desktop app
> (`apps/desktop`) has been removed. The macOS client is now the native Swift
> app in `apps/native`, and the session backend lives in the standalone
> `crates/` Rust workspace. References below point at the native + crates code;
> the on-disk contract now lives under `~/.unpeel`.

## North Star: Self-Hosted Cursor Alternative

> Full plan: `docs/MASTER PLAN.md`. This is directional — parts of it are not
> built yet — but new remote/mobile/host work should move *toward* this shape,
> not away from it.

The main goal: **run great on your own machines, remote-controlled from
Macs, iPads, and iPhones.** Unpeel is a **self-hosted Cursor alternative**: a
fleet of CLI agents you run and steer from anywhere, for *any* task, on
**hardware you own** (nothing leaves your machines). Same mobile-first
experience as Cursor's app — launch/steer agents from a phone, get notified,
review results, hand work between devices — but on Unpeel's thesis:
provider-agnostic, terminal-native, and **never a code IDE** (the Product
Philosophy guardrails above are hard constraints on this direction — the
review surface is **screenshots / demos + terminal + transcript**, *never*
diffs / PR-merge / file-tree).

The model is **hosts and controllers** — as roles, not app modes
(D1 amended 2026-07-23; headless hosts added 2026-08-07):

- A **Host** owns sessions. Two kinds, one protocol:
  - the **Mac app** — *every* desktop install is a host; there is no
    controller-only install or setup fork;
  - a **headless host** — `unpeel` (the terminal UI) on a Mac or a **Linux
    server**, hosting sessions and serving controllers with no desktop app
    present. On a server the TUI *is* the terminal server.
- A **Controller** drives a host remotely: the **iPhone / iPad app** (always a
  controller), or **another Unpeel desktop app** via the sidebar **Host
  picker** ("Local" is the default, "Share This Mac…" exposes this Host's
  one-time code, and "Add Host…" pairs another), which reaches both kinds of
  host.

**Selecting a remote host in the picker scopes the whole UI to it** — same
sidebar/terminal/verbs, sessions live on the host. Setup asks nothing new; a
"controller-only" presentation (hide Local) is at most a later optional
setting. There is no server *product*, no cloud tier, no
multi-tenant/workspace anything: a Linux host is **your** box running your
agents, self-hosted in the same sense a Mac host is. Nothing is shared, sold
as a service, or run by us.

> **Status (2026-08-11):** headless hosting is real but incomplete. The TUI
> hosts sessions, pairs phones, and serves the `/mobile` protocol app-lessly
> today (see `crates/unpeel-tui/tests/`, cases `standalone` and `mobile`).
> It now supervises `__remote__` (the TLS/WSS server) and has a
> conformance-tested Rust relay uplink. Clean Linux containers now pass the
> two-binary build/install/basic hosted-PTY proof (aarch64 native, x86_64
> emulated). The Host-side SSH stdio gateway and reusable system-SSH
> Controller connection now exist; their fake-SSH/real-gateway process proof
> covers multiplexing, blocked writes, reconnect/cursor resume, and ambiguous
> effect non-replay. Opaque connection-generation binding prevents a later
> call from crossing an idle SSH process replacement without a newly accepted
> bootstrap. A developer-only read-only bootstrap probe with Session listing
> consumes the production SSH connection. A transport-neutral remote backend
> now validates and pins typed bootstrap, stages bounded output pages, resumes
> exact committed cursors, and sends generation-bound terminal write,
> desktop-fit/clear, and mark-read effects without automatic replay. Its real
> gateway process proof keeps a blank Controller home untouched. The TUI now
> consumes that backend through strict `unpeel --host ssh://HOST` dispatch:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unpeel-com/unpeel](https://github.com/unpeel-com/unpeel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
