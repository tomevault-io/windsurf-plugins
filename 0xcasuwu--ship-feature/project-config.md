---
trigger: always_on
description: >
---


# Ship Feature

Autonomous 4-agent pipeline: Planner → Coder → Tester → Reviewer, followed by an
orchestrator-run **live-proof loop** that iterates the feature against the real
e2e surface (devnet, staging, local app, whatever your project has) until green.
Every agent is pre-seeded with the full codebase context for the repo(s) in scope, plus the
reference files that match the feature.

**Trigger:** `/ship <feature description>` or any request to implement + ship autonomously.

---

## Pipeline overview

```
Step 0: Live recon → safety preflight → load context docs → select reference files → build REPO_CONTEXT
    ↓
Step 1: Planner (strongest model)   → .pipeline/spec.md
    ↓
Step 2: Coder (mid-tier model)      → .pipeline/changes.md
    ↓
Step 3: Tester (mid-tier model)     → .pipeline/test-results.md
    ↓
Step 4: Reviewer (strongest model)  → .pipeline/review.md
    ↓
Step 4.5: Live-Proof Loop (orchestrator) → iterate-until-green → .pipeline/live-proof.json
    ↓
Step 5: Report VERDICT to user   (SHIP (LIVE-PROVEN) / SHIP (UNIT-PROVEN) / NEEDS WORK / BLOCK)
```

---

## Reference files

Reference files live in `references/`. They DEEPEN correctness; they never GATE it — an
orchestrator that reads ONLY this SKILL.md still runs a correct (if shallower) pipeline. Step 0d
decides which to inject; Step 0e pastes each selected file verbatim under its sentinel, exactly
like `REPO_CONTEXT`.

- **`references/debugging-doctrine.md`** — fast reproducer first; A/B both directions; never
  diagnose through an unverified read layer; read post-state before theorizing; monitor failure
  signatures; peel one layer per run. Domain-agnostic. → Tester + the Step 4.5 live-proof loop.
- **`references/e2e-spec-engineering.md`** — ONE stapled spec, cold-boot, structural assertions,
  locale-agnostic selectors, toast-as-evidence-capture, multi-actor testing, report JSON, machine
  safety. → Coder + Tester on app repos with a browser e2e surface.
- **`references/example-domain-brain.md`** — a worked EXAMPLE of a domain-specific reference
  (the author's, for a Bitcoin metaprotocol). Write your own for your domain (see "Adapting
  this skill" below) and inject it into all four agents when a feature touches that domain.

---

## Adapting this skill to your project (do this once)

This skill ships with placeholders where project knowledge belongs. Fill in three things:

1. **Repo table** (in "Repo Context Templates" below): list your repo(s), their root paths, and
   one line on what each is. Step 0a re-verifies this against disk every run anyway.
2. **Repo context template(s)**: for each repo, write a `=== <REPO> — MANDATORY CONTEXT ===`
   block holding the invariants an agent must never violate — the "never do X" rules, the
   source-of-truth rules, the key file paths, the safety rules. Mine your CLAUDE.md, past
   incident writeups, and code-review feedback. Specifics beat paraphrase: "never call
   `fetch(rpc)` directly — use `lib/rpc.ts` wrappers" is useful; "follow good practices" is not.
3. **Domain brain(s)** (optional but high-leverage): if your project has a domain with
   non-obvious failure modes (a blockchain protocol, a distributed system, a hardware target),
   distill the hard-won facts into a `references/<domain>-brain.md` modeled on
   `example-domain-brain.md`. Every rule should state its WHY (the real burn that produced it)
   so a zero-context agent trusts it.

---

## Step 0 — Setup, Live Recon & Context Preloading

This step is mandatory and must complete before any agent is spawned.

### 0a — Live repo recon (REPLACES blind trust in the repo table)

The repo table below is a recon-refreshed **default**, not gospel — branches and primary repos
move. For each candidate repo, verify on disk:
`git -C <path> rev-parse --abbrev-ref HEAD` (branch), `git -C <path> status --porcelain | wc -l`
(dirty count), `ls <path>` (exists). BUILD the working table fresh from these; override the
defaults on any mismatch. Ask the user if the repo(s) in scope are unclear — a single feature may
touch multiple repos.

### 0b — Concurrency & safety preflight

Run these BEFORE touching any repo:

1. **Concurrent-session detection.** Check mtimes of `.pipeline/*` and dirty trees (optionally
   `ps aux | grep claude`). Two sessions in the same checkout fight over HEAD and kill each
   other's browsers — isolate in a `git worktree` if detected.
2. **Checkpoint WIP first.** If the target repo is dirty or on a non-default branch, CHECKPOINT
   the working tree on a branch/commit BEFORE editing — never disturb someone else's uncommitted
   work. When in doubt, run the pipeline in a separate worktree.
3. **Resource gate.** If the live-proof surface is memory-heavy (an in-browser VM, a local
   chain, a big test harness), check free memory BEFORE any live run (on macOS:
   `vm_stat` free pages > 5000). Below the floor, the live loop is DEFERRED and the verdict is
   UNIT-PROVEN — a heavy run on a starved machine can swap-death freeze the host.
4. **One heavy harness at a time.** Never two memory-heavy harnesses in parallel; never run the
   live e2e alongside a manually-booted instance of the same surface.
5. **Port etiquette.** Do not assume the default dev port. Check who owns candidate ports
   (`lsof -ti:<port>`) and drive the spec via an env var (e.g. `PLAYWRIGHT_BASE_URL`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xcasuwu/ship-feature](https://github.com/0xcasuwu/ship-feature) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
