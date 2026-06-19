---
trigger: always_on
description: >
---


# all-deploy

Detects, audits, and deploys projects to the right host with a confirmed preview → prod flow. The unique value is **detect → audit → route to the right CLI**, not wrapping the CLI.

## Configuration (edit these lines to change default behavior)

```
CONFIRMATION_MODE: ask_at_start
  # ask_at_start (default): the skill asks at the start of each cloud deploy
  #                         whether to run full-auto or step-by-step.
  # full_auto:              skip the question. Audit → preview → prod with a
  #                         5-second ESC window before prod.
  # always_ask:             skip the question. Require explicit "yes" before prod.
VISUAL_VERIFY: false           # true → screenshot preview via chrome-bridge-automation (frontend only)
ALLOW_DIRTY_TREE: false        # true → skip the "HEAD is clean" audit check
```

The default `ask_at_start` works for both solo authors and community users — the mode choice moves to runtime. Pinning to `full_auto` or `always_ask` is only useful when wrapping the skill in automation that can't answer the question interactively.

## Hard Rules (never violate — these are the safety contract)

1. **Never bypass or soften the audit.** The audit is the only gate between the user's intent and live infra in `full_auto` mode. If a check cannot run, that itself is a block, not a reason to skip.
2. **Never deploy to prod before a successful preview with a 2xx/3xx health check.** Preview → summary → prod. No prod without a green preview in the same session.
3. **Never commit, print, or log secrets.** If a secret is found in tracked files, halt and guide removal + history rewrite as a separate user-approved step. Secret values never appear in summaries, logs, or commit messages.
4. **Never auto-install or auto-authenticate third-party CLIs.** Hand the user the command with the `!` prefix (e.g., `! vercel login`, `! railway login`).
5. **Never wrap a target CLI in a custom script that hides flags.** Reference files document real commands the user can read and verify.
6. **Never modify code to make audit pass without showing the diff first.** `.gitignore`, `.env.example`, Dockerfile scaffolding, port-binding fixes — all shown and approved before applying.
7. **Never deploy from a dirty working tree** unless `ALLOW_DIRTY_TREE: true` is set. Uncommitted changes complicate rollback.
8. **Respect "wait."** Between preview and prod, any user utterance resembling hesitation (wait/hold/stop/not yet/abort) aborts the prod step cleanly.

## Two Modes

Decide up front from the user's phrasing:

- **Cloud deploy** (default) — Phases 0 through 6 below. Triggers: "deploy", "ship", "push to prod", "get it online", "/all-deploy".
- **Run locally** — Phases 0, 1, 2, then a local-run step. Triggers: "run it locally", "corre esto localmente", "/all-deploy local", "just run it", or `--local`.

In **run locally** mode, run a **scoped audit** — only secrets-in-tracked-files, start-command-exists, and sane port binding. Skip: git-remote, clean HEAD, `npm audit`/`pip-audit`, `.env.example` completeness, and env-var delivery (no remote to deliver to). Then execute the detected start command (`npm run dev` / `uvicorn main:app --reload --host 0.0.0.0 --port $PORT` / `docker compose up` etc.) in the foreground, streaming output via `Monitor`. If the user asks to also expose the local server, chain into `references/targets/cloudflared-tunnel.md`. Everything else in this file concerns cloud deploy.

## Cloud Deploy Workflow

### Mode selection (runs before Phase 0 in `ask_at_start` mode)

Resolve the effective `CONFIRMATION_MODE` for this run:

1. **If the user's invocation contains an explicit mode marker**, adopt it and skip the question:
   - `full_auto` triggers: "auto", "full auto", "auto deploy", "deploy auto", "/all-deploy auto", "yolo".
   - `always_ask` triggers: "step", "step by step", "one at a time", "confirm each step", "paso a paso", "/all-deploy step".
2. **Else if `CONFIRMATION_MODE` is pinned to `full_auto` or `always_ask`** in the config block above, use that and skip the question.
3. **Otherwise (`ask_at_start`, default)**, ask the user once, before Phase 0 runs:

   > *"Before we start — how should I run the deploy?*
   >
   > *1. Full auto — I run audit → preview → prod promotion in sequence. Prod fires after a 5-second ESC window.*
   > *2. Step by step — I stop for your OK between audit, preview, and prod.*
   >
   > *Reply 'auto' or 'step'."*

The chosen mode scopes to this run only — a future invocation asks again (unless the user explicitly opts in via trigger phrase or the config block is pinned).

"Wait" / "stop" / "hold" at any point still aborts the prod step in both modes (Hard Rule 8). The difference is whether you get a 5-second ESC window (full_auto) or an explicit "yes"-required pause (always_ask) before the prod command runs.

### Phase 0 — Prerequisites (target-independent)

Check in order. Each failure halts with a `!` command for the user.

1. **Git repo.** Run `git rev-parse --is-inside-work-tree`. If not a repo, halt unless mode is `run locally` (which doesn't require git).
2. **Git remote** (cloud deploy only). Run `git remote -v`. If no remote is configured, halt *unless* the target will be `docker-vps` (deployable without a remote). This second clause is re-evaluated after Phase 3.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hainrixz/all-deploy](https://github.com/Hainrixz/all-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
