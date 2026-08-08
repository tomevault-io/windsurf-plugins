---
trigger: always_on
description: Read [`CONTRIBUTING.md`](CONTRIBUTING.md) for the stack contract (what-lives-where, the ABI
---

# CLAUDE.md — instructions for all agents working in this repo

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) for the stack contract (what-lives-where, the ABI
rule) and [`DIARY.md`](DIARY.md) for why. This file is the **operational protocol every agent
must follow when building/testing containers**, so we stop wasting each other's GPU time.

## Branch / worktree protocol (MANDATORY — read FIRST)

**NEVER do feature work in the shared `main` checkout — a dedicated worktree is REQUIRED.**
Multiple agents work different features in this repo at the same time. Two failure modes, both
chaos:
1. Editing on `main` directly → uncommitted changes collide and entangle (e.g. one agent's edits
   layered on another's uncommitted refactor — impossible to split cleanly). Already bit us.
2. `git switch`/`checkout -b` in the *shared* checkout → you change the branch out from under
   every other agent working in that same directory. Switching branches underneath each other is
   just as much chaos as sharing a dirty `main`. **Branch-switching in the shared checkout is NOT
   an acceptable substitute for a worktree.**

- **Before touching code, create your own feature-branch *worktree* (a separate directory)** and
  work entirely inside it — never `git switch` the shared checkout:
  ```
  git worktree add -b feat/<short-topic> ../vllm-gfx1201-<short-topic> main
  cd ../vllm-gfx1201-<short-topic>
  ```
  Each agent/effort gets its own worktree dir, so no one's branch or working tree moves under
  anyone else. This is mandatory, not preferred.
- **One worktree/branch = one concern.** Do not bundle an inherited uncommitted refactor with your
  own change; if you find a working tree already dirty with someone else's WIP, STOP and flag it —
  don't commit it under your change.
- **Commit/push only when asked.** When you do, it is on your feature branch, never `main`.
- Read-only analysis, profiling runs, and container builds are fine from the shared checkout; it's
  *source edits* that must live in your own worktree.

## GPU sharing protocol (MANDATORY — no human booking manager)

The box has **two** discrete gfx1201 compute cards: **GPU 0** (RX 9070 XT, 16 GB) and **GPU 1**
(RX 9070, 16 GB). ROCm device **2** is the Ryzen iGPU (gfx1036) — it drives the display and is
**never** a compute target. Multiple agents share these two cards.

**The lease interface is its own canonical repo now: [`/home/pat/code/gpu-lease`](/home/pat/code/gpu-lease)** —
the single place ALL GPU leasing (local + cloud) is managed and monitored from. It is on `$PATH`
(via `lease install`), so invoke it by bare name — **never** by a `scripts/…` path. If `gpu-lease`
isn't found, run `/home/pat/code/gpu-lease/bin/lease install` then open a new shell.

**Do NOT ask a human "can I have the GPU?" and do NOT just check `rocm-smi` and hope. EVERY GPU
workload — `docker compose` serve/bench, a raw `python`/pytorch probe, anything that touches a
card — MUST be launched through `gpu-lease` (aka `lease local`).** It is an `flock`-based arbiter:
the lock is held by the launched process and dies with it, so a crash/Ctrl-C auto-frees the cards
with no stale "reserved" state and no janitor. This *is* the booking manager — there is no human in
the loop.

```
# Foreground job (bench, probe, smoke test) on whichever card is free; blocks until one is:
gpu-lease -n 1 -- docker compose --profile bench run --rm bench
gpu-lease -n 1 -- bash -c 'source /app/.venv/bin/activate && python my_probe.py'

# Long-lived detached server; blocks until enough cards are free, then returns immediately and
# keeps the lease alive until the CONTAINER stops (not until this command returns):
gpu-lease -n 2 --detach --name serve35b -- docker compose --profile serve up -d --build
gpu-lease -n 1 --detach --name zaya     -- docker compose --profile zaya  up -d

# See who holds what (truth = flock state, self-healing):   gpu-status   (or: lease status)
# Unified local+cloud monitoring dashboard (CPU-only, no lease):  lease console  → http://127.0.0.1:8787
```

The flock lock dir is now box-global at `$XDG_CACHE_HOME/gpu-lease/locks` (override `GPU_LEASE_LOCKDIR`),
so every worktree coordinates on the same files without naming a repo. Cloud leases (RunPod / Hot
Aisle / Vultr / …) run through `lease cloud …` / `cloud-lease` from the same repo — see its README.

Rules:
- **`-n` = HOW MANY cards, NOT which card.** It is a count: `-n 1` = one card, `-n 2` = both. TP=2
  serve/bench/zaya → `-n 2`; a single-card model/probe → `-n 1` (the default). **There is no
  "pin a specific GPU" flag and you never need one** — the arbiter auto-assigns the lowest free
  card and injects `ROCR/HIP_VISIBLE_DEVICES` for you. Do NOT pass `-n 2` thinking it selects
  "card #2": that leases BOTH cards and starves every other agent. Want a single card? `-n 1`,
  always — let the arbiter choose which one.
- **`--detach` for any `up -d`** (the holder binds the lease to container lifetime). Foreground jobs
  (`run --rm`, a script that blocks) need no flag — the lease releases when they exit.
- **Let it block (the default).** Waiting *is* the coordination — don't poll `rocm-smi` and don't
  fall back to asking. Use `--nowait` only when you genuinely want to skip if the box is busy, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patcarter883/rdna4-vllm](https://github.com/patcarter883/rdna4-vllm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
