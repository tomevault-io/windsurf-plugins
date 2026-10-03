---
trigger: always_on
description: Canonical repository instructions for Codex, Claude Code, and other coding agents. Keep this file
---

# AGENTS.md — stellar-raven-codemode

Canonical repository instructions for Codex, Claude Code, and other coding agents. Keep this file
short, current, and operational. Put architecture, research, history, and task runbooks in the
linked docs and skills instead of accumulating them here. Human contributors start with
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## Project and source-of-truth map

This is a Cloudflare Workers MCP server exposing `search` and `execute` over Lumenloop, Stellar
Light/Scout, Stellar Docs (Algolia), and selected ecosystem skills. Model-authored JavaScript runs
in a networkless Dynamic Worker; host adapters own all service traffic, policy, and secrets.

- Read `PLAN.md` for current product scope and status, then `ARCHITECTURE.md` for the implemented
  request, catalog, scoring, sandbox, and auth design.
- Use `README.md` for connection and local setup, and [`docs/operations.md`](docs/operations.md)
  for operator procedures.
- Use `research/` for dated evidence and design context; it is not an instruction layer.
- This repo is self-contained, including usage collection and reporting under `usage/`. Owner-only
  private folders exist outside the repository; `usage/README.md` names them and their roles. Never
  depend on them, and never commit production counts here.
- Use `.agents/skills/<name>/SKILL.md` for repeatable task workflows. `.claude/skills` is the
  committed symlink to the same canonical directory.
- Use `.agents/TODO.md` for the own-repo work queue, priorities, and open owner decisions, and
  `.agents/rounds/` for round ledgers. `.agents/README.md` says which note belongs where.
- `CLAUDE.md` imports this file. Do not duplicate shared rules there.

## Commands and verification

- Install reproducibly: `npm ci`. The `prepare` script installs a pre-commit hook that scans
  staged files for secrets.
- Fresh clone: follow "Run locally" in `README.md`. `npm run typegen` needs the placeholder
  `.dev.vars` first; without it, `Env` has no secret members and `npm run typecheck` fails.
- Baseline validation for code changes: `npm run typecheck`, `npm test`, and `npm run build`.
  `npm test` excludes `test/smoke/**`; add `npm run test:smoke` when touching `src/executor` or
  `src/demo` — it is the only lane exercising those paths against the assembled worker
  (unit tests import the modules directly; CI always runs both).
- CI (`.github/workflows/ci.yml`) also runs `npm run eval:selftest`,
  `npm run eval:qa:lint -- --stale --enforce-floors`, `npm run eval:qa:register -- --check`,
  `npm run improvements:lint`, `node scripts/check-pin-review.mjs --base <ref>`,
  `npm run eval:routing -- --gate`, and a generated-artifact sync check. Run the matching command
  when you touch `eval/`, `improvements/`, `catalog/`, or `ecosystem-skills/`.
- Run the narrowest relevant eval or maintenance command in addition to the baseline; the selected
  skill defines the exact gate for eval, drift, golden-truth, improvements, and observability work.
- Scan before committing: `npm run secrets:scan -- --tree`.
- Do not start a second Wrangler process. Reuse the pane that already runs `npm run dev` (manual
  testing) or `npm run dev:eval` (eval lanes; see `run-evals`), and read its bound URL from that
  pane's output.
- Generated outputs are rebuilt by their `package.json` scripts, never edited by hand.

## Coordination

The owner runs agents under Herdr, and the rules in this section and the next describe that setup.
Without Herdr, work in one session. State in the pull request that the independent review is
outstanding; the owner runs it.

- Agents, panes, and worktrees run under Herdr. Use the global `herdr` skill for the CLI contract;
  it is the authority on command syntax, lifecycle states, and ID handling. Confirm
  `HERDR_ENV=1` before any control command, and read IDs out of JSON responses rather than
  predicting them.
- Spawn a reviewer or a parallel lane by splitting a pane from your own —
  `herdr pane split --current --direction <right|down> --cwd "$PWD" --no-focus`; `--direction` is
  required — then `herdr agent start <name> --kind <codex|grok|claude> --pane <id> -- <cli args>`.
  Name the model and effort explicitly after `--`. Wait with `herdr agent wait <name>` or
  `herdr agent prompt … --wait`; do not poll.
- **Pane and agent ownership is recursive:** control only the pane you occupy and panes you
  split yourself. Never close, interrupt, restart, send input to, rename, move, focus, resize, or
  take over a parent, a sibling, an unrelated pane, or another agent's descendants. Apply the same
  rule to every sub-agent. Idleness, staleness, a completed handoff, or a request to clean up does
  not transfer ownership: leave a pane you do not own alone and ask its owning agent to reconcile
  it. Record the pane IDs you create; unknown provenance means not owned. If the owner is unknown
  or unavailable, ask the user for an explicit exception naming the exact target; never adopt it.
- `herdr agent read` cannot recover output that scrolled off an alternate screen. For any reviewer
  whose findings matter, have the agent write them to a Markdown file and reply with only the path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stellar-experimental/stellar-raven](https://github.com/stellar-experimental/stellar-raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
