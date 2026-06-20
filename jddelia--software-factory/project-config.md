---
trigger: always_on
description: Instantiate and operate an adaptive Codex software factory for substantial software projects. Use when Codex should ask or infer a factory work mode, including Safe MVP mode for thin real vertical slices, set up persistent factory goals, configure Principal Executive/user-involvement pattern, role topology, ledger topology, baton sizing, verification cadence, acceptance tiers, reviewers, manager/user-liaison intake, thread cleanup, stop/resume controls, handoffs, monitors, commits, release gates
---


# Software Factory

## Outcome

Create a configurable software factory that matches the project's risk, maturity, and delivery target:

- an explicit factory configuration;
- an optional Principal Executive / overseer for user partnership, topology changes, and factory-level intervention;
- one executive with final product and quality authority;
- either executive-as-ledger, separate-ledger, or passive-fallback topology;
- one active writer per worktree by default;
- optional Manager/User Liaison for packaging user feedback into Executive Briefs;
- optional parallel read-only reviewers;
- optional thread cleanup after evidence is captured;
- explicit stop/resume controls for safe pauses, hard stops, monitor handling, and resumable handoffs;
- explicit permission, model, and blocker policies so worker threads do not stall on predictable tooling friction;
- explicit tool-call budgeting and thread-read policies so controllers do not stall on oversized or schema-invalid inspection calls;
- adaptive active-actor polling so controllers do not waste tokens watching normal Builder progress;
- capability preflight before launch so missing thread, browser, git, env, network, or approval capabilities are visible;
- baton handoffs with acceptance tiers;
- verification cadence that can favor safe MVP delivery, velocity, balance, strictness, release, or recovery;
- commits only for accepted work unless the user explicitly delegates otherwise.

This skill is broadly applicable to software projects. Do not hard-code project-specific rules. Infer project-specific invariants from the repository, user request, specs, tests, and risk surface.

## Runtime Fit

Prefer the Codex app for this skill. It is optimized for Codex primitives such as delegated worker threads, tool permissions, browser QA, persistent goals/checkpoints, bounded thread reads, and long-running handoffs.

Use other agent runtimes only when they provide comparable primitives or when the factory is intentionally operating in manual/single-thread mode through the seeded docs and baton protocol. When a runtime lacks a needed primitive, record it during capability preflight and downgrade topology instead of pretending the factory can run fully autonomously.

## Alignment Before Instantiation

Before creating a factory, align on the minimum configuration needed for useful autonomy. Prefer a short, human-readable intake over exposing raw config.

Ask the user which `work_mode` to use unless the request clearly implies one. Keep the question short and include concise descriptions:

- `balanced`: normal feature work, good quality without heavy ceremony.
- `velocity`: bigger vertical slices, focused checks, periodic full gates.
- `safe_mvp`: the thinnest real product slice; cut breadth and polish while preserving hard safety and verification invariants.
- `strict`: smaller batons, deeper review, full gates for high-risk work.
- `prototype`: move fast to prove the experience; hardening is tracked separately.
- `release` or `recovery`: ship readiness, or cleanup of messy/broken state.

Ask at most three short questions by default:

1. Which work mode should I use?
2. What is the target outcome: end-to-end build, specific feature, release readiness, recovery/cleanup, or another objective?
3. How should user involvement and side feedback work?

For the third question, use plain options:

- Principal Partner: the current user-facing thread acts as a top-level partner/overseer while the Executive Ledger drives the factory.
- Direct Executive: the user talks directly to the active Executive/Ledger.
- Hands-Off: the factory runs with checkpoint summaries.

Then ask feedback handling only when the project is long-running, the user expects to comment while work continues, or the answer is unclear:

- No Manager: keep the active Executive lean; user feedback goes directly to the chosen contact point.
- Feedback Manager: package user side feedback into briefs so the Executive stays focused.
- Always-On Manager: continuously monitor user feedback and factory state, then brief the Principal or Executive.

Do not ask if the user's request already gives enough signal. Infer `safe_mvp` from "few hours", "ship the MVP", "thin real slice", "move fast but safely", or "working core flow now"; infer `velocity` from "move fast" or "iterate quickly" when the user still expects normal integration breadth; `strict` from "production-grade", "no shortcuts", "all tests", or safety-critical work; `discovery` from "review/understand"; `recovery` from broken/colliding state; `release` from deploy/ship/merge readiness. Infer `Principal Partner` for substantial long-running builds unless the user wants a single direct thread or hands-off automation.

## First Move


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jddelia/software-factory](https://github.com/jddelia/software-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
