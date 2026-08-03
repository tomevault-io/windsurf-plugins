---
trigger: always_on
description: auto-gtm is a Claude Code / Codex plugin: a family of GTM automation tools, one skill per scenario × platform — topic scouting (your PRs + today's hotspots), X post and reply drafts, Reddit subreddit-finding and reply/post drafts. Every tool stops at drafts and analysis — it never publishes, comments, or performs any platform write on its own. The wedge vs neighboring tools: GTM material is grounded in what you actually shipped and what's being discussed today, drafted in a voice you chose — not
---

# auto-gtm

auto-gtm is a Claude Code / Codex plugin: a family of GTM automation tools, one skill per scenario × platform — topic scouting (your PRs + today's hotspots), X post and reply drafts, Reddit subreddit-finding and reply/post drafts. Every tool stops at drafts and analysis — it never publishes, comments, or performs any platform write on its own. The wedge vs neighboring tools: GTM material is grounded in what you actually shipped and what's being discussed today, drafted in a voice you chose — not generated from thin air.

## Layout

- `skills/<name>/SKILL.md` — shipped skills, the product surface. One GTM tool = one skill directory; helpers live beside it under `scripts/`, `references/`, `templates/`.
- `.claude-plugin/` + `.codex-plugin/` — plugin manifests for the two hosts; keep both in sync on every skill add/rename/version bump.
- `docs/design/` — architecture and the why, per tool. Start at the index, then the tool you're touching. (Create as the design solidifies; keep the index current.)
- `docs/design-harness/` — the evidence board (sources → ideas → output) behind positioning and design calls; operate it via the design-harness skill, never by hand-editing its state.
- `docs/plans/` — per-change implementation plans (working artifacts, exempt from the design-doc style rules).
- `docs/TODO.md` — tracked follow-ups not yet on the roadmap. Keep current in real time (see rule below).
- `README.md` — install/quickstart runbook; update it in the same change whenever install, update, or usage flow changes.

## Task lifecycle — the fixed order for every non-trivial change

Plan → sync → docs → tests → skill/code → verify → commit → docs/index sweep → green CI. Trivial one-line changes skip the plan; nothing skips the order. A change that arrives out of order is incomplete.

1. **Plan.** Non-trivial changes start with a plan in `docs/plans/`. The plan's first unit updates the relevant `docs/design/` doc; every unit places acceptance criteria before implementation.
2. **Branch, isolate, sync.** Bind each change to exactly one explicit task on its own branch, developed in its own worktree — created with Claude Code's own worktree tooling (the built-in worktree feature), which places it under `.claude/worktrees/<branch>` (`.claude/` is git-ignored); never hand-create one with raw `git worktree add`, and never develop on `main`. Before developing, `git fetch` and rebase onto the latest `main`. Land via feature branch + PR; no direct pushes to `main`.
3. **Docs first.** Read, then update, the relevant `docs/design/` doc(s) before touching any skill or script — pin down behavior boundaries, interface contracts (skill trigger, inputs, outputs, stop-at-draft line), and acceptance criteria. Never touch the skill first.
4. **Tests next.** Where a tool has executable parts (`scripts/`), write the failing test first in the matching `tests/` file, use fixtures rather than live platform APIs, and assert relationships/invariants — never hardcoded values that break when an upstream feed changes. For prompt-only skills, the test is a written acceptance checklist in the plan: input transcript → expected draft properties.
5. **Skill/code.** Write the SKILL.md / scripts that satisfy the criteria.
6. **Verify end-to-end.** Install the plugin locally and run the skill against a real session/feed; confirm the full flow (trigger → distill → draft → stop) — not just unit tests. A skill shipped without an end-to-end run is incomplete.
7. **Commit.** Run relevant tests before each commit; commit at every green point and `git push` after every completed task — progress must never exist only on this machine.
8. **Sweep docs & indexes.** Confirm whether the change needs updates to `docs/design/` indexes, `README.md`, and both plugin manifests — update them in the same change. Version bumps are patch-only (`0.2.0 → 0.2.1`) unless explicitly requested otherwise.
9. **Drive CI green.** After opening the PR, watch CI; on any failure, locate, fix, and push immediately — repeat until every required check passes.

## Working rules

- **Docs are top-level design only.** Describe what a tool does and why — never how. No pseudocode, no code snippets, no concrete data, values, or magic numbers. Name skills and objects — never functions, constants, or file paths; that detail lives in the skill/code. Two carve-outs: architecture diagrams stay; and the setup runbooks (README install/quickstart) keep the literal commands, since the command is the deliverable there.
- **Design docs are ruthlessly concise** — every sentence earns its place. No filler, no hedging, no restating what a SKILL.md, a diagram, or another doc already says. One fact lives in exactly one place; cross-link instead of repeating. When you edit a doc, leave it shorter than you found it unless you added a genuinely new idea.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigerless-labs/auto-gtm](https://github.com/tigerless-labs/auto-gtm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
