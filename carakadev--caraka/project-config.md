---
trigger: always_on
description: Instructions for coding agents working on this repository. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md) first; everything here applies to both.
---

# AGENTS.md

Instructions for coding agents working on this repository. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md) first; everything here applies to both.

## What this project is

Caraka is a bridge from a chat app to the coding agent already installed on the user's machine. Telegram came first, Discord landed in v0.5, and WhatsApp in v0.6; all three speak the same `Channel` contract. It has no reasoning loop, no execution tools, no model provider, and no plugin marketplace, on purpose.

Before writing code, read `docs/blueprint.md` and the phase you are working in from `docs/roadmap.md`.

## The rule that governs every change

> **Does the coding agent already do this?** If yes, we do not build it.

Proposals that add an agent loop, execution tools, a model abstraction, or a plugin registry will be declined however well implemented.

Two more constraints shape every review:

- **Complexity budget.** A new feature must either remove something or keep the core under ~8,000 lines. v1.1 broke this rule and the number is recorded rather than the ceiling moved: `src/` measured 8,349 lines on 8 August 2026, against 7,880 at v1.0. A simplification pass returned 73 lines and stopped where a normalised block scan stopped finding repetition. Raising the ceiling because we crossed it is how a budget stops being one.

  The debt grew again rather than being paid. On 10 August 2026 `src/` measures 8,498 lines, +149 from rewriting `src/memory/titen.ts` against a live Titen, making `caraka doctor` prove a credentialed call, and breaking a tie in `activeGrant` that let two trust windows opened in the same millisecond be chosen between at random. Most of it is not logic: the adapter's header block records the exact rejection each wrong field caused, because the previous version was 111 lines that agreed with a document and with nothing the server accepts, and six of the six lines the tie-break cost are the comment explaining why one word of SQL is there. Comments that stop a wrong shape from being written a second time are the last thing this budget should buy back.

  On 13 August 2026 `src/` measures **8,808 lines**, 808 over the ceiling. `workspace-dari-chat` added 262 of them, measured against the 8,546 the tree held when it started, six above the number the line below records. The estimate written into its spec before the code was ~100, and the gap is where the four preconditions turned out to live: a workspace path that is canonical where it becomes a grant key, a `caraka trust` that refuses a path no config names, a session slug that no longer resolves to the first workspace and inherits its trust window, a containment predicate `docs/security.md` §7 had promised since v1.0 with nothing behind it, and a `/lock` that stopped answering "no window is open" while every window stayed open. The feature on top of them is the path form in the operator's DM and the signed card that writes the entry. Fourteen of the lines are seven catalog pairs, and the comments carry the readings that made three earlier readings of these paths wrong. No deletion paid for it: each of the four candidates — the shared fetch-with-retry, `Channel.getMe()` with no caller in `src/core/`, the twin PRAGMA scans, the three memory command openers — belongs to another concern, and a PR that fixes a bug and refactors is two PRs. The ceiling stays ~8,000.

  On 13 August 2026 `src/` measured **8,540 lines**, +42 from `spawn-windows`. It bought a crash: the one `spawn` in the tree with no `"error"` listener, which Node throws and which ended `caraka start` on any operating system before the fall to the CLI driver could run, and a `resolveCommand` that answered "exists on disk" where the question was "can be spawned" — on Windows those differ, and the npm shim it returned was the file that cannot. Four things went out with it: the `node_modules/.bin` branch, the second PATH walk in `discovery.ts`, the `realpathSync` that undid the first one's answer, and a `ponytail:` comment whose upgrade path CVE-2024-27980 had already closed. Eighteen of the 42 lines are comment, holding the libuv and npm facts that made two earlier readings of this bug wrong. The ceiling stays ~8,000, and the next feature owes 540 lines or a removal.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CarakaDev/caraka](https://github.com/CarakaDev/caraka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
