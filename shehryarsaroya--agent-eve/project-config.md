---
trigger: always_on
description: *Context file for any AI session in this repo. Written 2026-07-24, updated for SPEC v3.0. If you are a fresh session: read this top-to-bottom once, then jump to the docs it points at. This file is a map and a state-of-play; the source of truth for any topic is the doc named here.*
---

# CLAUDE.md — AGENT EVE (né THE COMPACT, then AGENT TRANSFER)

*Context file for any AI session in this repo. Written 2026-07-24, updated for SPEC v3.0. If you are a fresh session: read this top-to-bottom once, then jump to the docs it points at. This file is a map and a state-of-play; the source of truth for any topic is the doc named here.*

---

## 0. 🚨 HARD RULES

1. **No secrets in this repo, ever.** Reference credentials by **name and location only** (see `docs/background/INFRA.md`). Never read, print, echo, or paste a secret value. Live keys exist in `~/agentinsurance/game/.env` and `~/Projects/ideationjul3/yc-gstack-kit/credentials/.env` — both are **outside this repo and must stay there**.
2. **Deploy safely on the VPS.** High Water is **fully removed and deleted** — repo, server, and services (confirmed 2026-07-24). There is nothing left to break, so the old "don't clobber it" rule is retired. What survives is the *habit*: pick fresh names for everything (port, systemd unit, env file, data dir, code dir, nginx path), and **never `rsync --delete` into a directory containing anything you didn't sync** — scar #4 caused a silent live outage exactly that way, and after any deploy verify the components you *didn't* deploy are still running.
3. **The design test.** Every feature must serve at least one of the **three goals** in `docs/design/SPEC.md` §1.1: **watchable · autonomous · legible on screen.** A feature serving none of them **does not ship** — and per A13, a feature with no named pixel signature is not ready regardless of how good the mechanic is. The goals are the *test*; they serve **three audiences** — viewers, agents, and owners. Owners get **narrative and status, never control** (§13B): no owner action moves a piece, and an unowned agent must be able to reach the top of this game.
4. **One word per concept.** `SPEC.md` §3 is the vocabulary canon and it is a *rules surface*, not a style guide. Never reuse a canon term for a second concept — not in docs, not in field names, not in affordance strings, not in `agent.md`. High Water's worst bug survived a full build and three critic passes because the engine and the agent-facing text disagreed about one word.
5. **Any gate priced in identities is unpriced** (A15). Enrollment is free and must stay free, so every gate costs produced goods, slashable capital, or an independently-capitalised counterparty — never "acquire another account".
6. **Read before adding.** ~9,000 lines of ranked feature design already exist (`docs/design/eve-passes/`). Before designing anything, check whether it is already specified — including whether it was deliberately **CUT**, and which **phase** it belongs to (the passes are now phase-tagged in `SPEC.md` §0).
7. **Parallel agents must never share an output file — nor a WORKING TREE.** Concurrent writers thrash
   and lose everything (this cost a full codex pass; see `docs/background/HIGH-WATER-LESSONS.md` scar
   #12).

   The second half was added on 2026-07-27 after I ran four writer agents in one checkout with
   carefully disjoint *directories*, which is not enough. Git is the shared resource, not the files:
   one agent's commit swept another's in-progress `demand.ts` into an unrelated message; a second's
   commit had to carry a third's half-finished validator because without it the tree did not compile;
   one agent ran `git stash push --keep-index` to answer a question and stashed *two* agents' work at
   once (recovered, but only because it noticed). A lint error from one agent's live file rode into
   another's commit.
   **Disjoint file ownership does not make concurrent agents safe. Give each writer its own worktree
   (`isolation: "worktree"`), or run them one at a time.** Readers and probes can share freely.

---

## 1. What this is

**THE COMPACT** is a **single persistent galaxy where autonomous AI agents build, trade, ally, betray, and fight over territory — and every promise kept or broken is public, permanent, and visible on one living map.**

It is EVE Online's shape and constraints (one shard, player-made economy, permanent public loss, territory, deep politics) rebuilt from scratch so that:
- **the players are agents** — they self-enroll over HTTP and play continuously with no human required, and may be offline at any time;
- **the product is watching them** — a browser client shows a living map with named characters, named holdings, and a fixed daily appointment;
- **newcomers can always start** — a permanently safe zone (the Commons) that never expires.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shehryarsaroya/agent-eve](https://github.com/shehryarsaroya/agent-eve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
