---
trigger: always_on
description: MSP automation platform built with FastAPI and React.
---

# Bifrost Integrations Platform

MSP automation platform built with FastAPI and React.

## Codex repo-local skills

This repository keeps repo-maintenance skills under `.claude/skills/`. Codex agents should treat these as repo-local skills when the task matches their descriptions:

- `.claude/skills/bifrost-issues/SKILL.md`
- `.claude/skills/bifrost-debug/SKILL.md`
- `.claude/skills/bifrost-release/SKILL.md`
- `.claude/skills/bifrost-testing/SKILL.md`
- `.claude/skills/bifrost-secupdate/SKILL.md`
- `.claude/skills/bifrost-secaudit/SKILL.md`
- `.claude/skills/bifrost-documentation/SKILL.md`
- `.claude/skills/bifrost-copilot-cowork-package/SKILL.md`

Do not treat `.claude/skills/bifrost-build` or `.claude/skills/bifrost-setup` as repo-local Codex skills here. Those are package/plugin-managed global Bifrost skills.

## Worktree rule (CRITICAL)

All changes — features, fixes, refactors, doc edits — must be made in a git worktree, never directly on `main` (or any shared branch) in the primary checkout. If a task is about to modify files and the current working directory is the primary `bifrost` checkout, stop and create or enter a worktree first. Use `EnterWorktree` (or `git worktree add .claude/worktrees/<name>`). The only edits permitted on `main` are ones the user explicitly requests be made there.

## Spinning up the dev environment and connecting

Use this whenever you need a running Bifrost instance to exercise — clicking around, screenshots, browser testing, or driving the API directly via the CLI. Always do this from the worktree, not the primary checkout.

1. **Boot the stack** from the worktree root:
   ```bash
   ./debug.sh up
   ./debug.sh status
   ```
   Capture the URL from `./debug.sh status`. Under netbird mode (default when `NETBIRD_SETUP_KEY` is set), the host-reachable URL is the full `http://bifrost-debug-<name>-<n>-<n>.netbird.cloud` form — the short `http://bifrost-debug-<name>` form is only resolvable from inside the mesh. Under port mode, the URL is `http://localhost:<port>`.

2. **Connect via the CLI** from an isolated scratch directory outside the repo (not the worktree, not `~`). This keeps `.env`/credentials out of the source tree and lets you install the API-matched CLI without disturbing the user's global `bifrost` install:
   ```bash
   mkdir -p /tmp/bifrost-cli-<name>
   cd /tmp/bifrost-cli-<name>
   python3 -m venv .venv
   .venv/bin/pip install --quiet --upgrade pip
   .venv/bin/pip install --quiet "<API_URL>/api/cli/download"
   ```
   The download endpoint serves the build matching the running API; installing this version avoids the version-mismatch warning that otherwise short-circuits subcommand output.

3. **Log in** inside the scratch directory using password-grant. This writes `.env` (with `BIFROST_API_URL`, `BIFROST_ACCESS_TOKEN`, `BIFROST_REFRESH_TOKEN`) so subsequent commands in this directory pick the tokens up automatically:
   ```bash
   ./.venv/bin/bifrost login --url <API_URL> --email dev@gobifrost.com --password password
   ```
   Default credentials are `dev@gobifrost.com` / `password` (MFA off) — password-grant works only because the dev stack has MFA disabled.

4. **Drive the API** with `./.venv/bin/bifrost <entity> <command> ...`. Use `--help` on any subcommand. Browser testing goes against the URL from step 1.

Tips:
- If a CLI command appears to succeed silently, run the matching `list` to verify — version-mismatch warnings can otherwise mask failures (do not pipe to `/dev/null` blindly).
- The netbird sidecar can transiently drop the peer; if connections start failing, re-check `./debug.sh status` and re-fetch the URL.
- For seeding entities to exercise a UI change (e.g. enough apps/agents to make a page overflow), use `bifrost apps create`, `bifrost agents create`, etc., from this same scratch venv.

## Technologies

-   **Backend**: Python 3.11 (FastAPI), SQLAlchemy, Pydantic, PostgreSQL, RabbitMQ, Redis
-   **Frontend**: TypeScript 4.9+, React, Vite
-   **Storage**: PostgreSQL (data), Redis (cache/sessions), RabbitMQ (message queue)
-   **Infrastructure**: Docker, Docker Compose, GitHub Actions for CI/CD

## Development Environment (CRITICAL - READ FIRST)

### Everything Runs in Docker

**Development happens inside Docker containers, not on the host machine.**

Start the development stack (per-worktree isolated):
```bash
./debug.sh             # Boot stack, print URL + login
./debug.sh status      # Show URL, login, mode (port or netbird)
./debug.sh down        # Tear down + remove volumes for THIS worktree
./debug.sh logs api    # Follow logs for one service
```

`./debug.sh` derives its Compose project name from the worktree path, so multiple worktrees can run debug stacks in parallel. URL and login are printed at the end of `up` (default: `dev@gobifrost.com` / `password`, MFA off).

The default mode allocates a free local port for the client (deterministic per worktree, in 30000-39999). If `NETBIRD_SETUP_KEY` is set in `~/.config/bifrost/debug.env`, the stack boots with a Netbird sidecar instead and is reachable at `http://<bifrost-debug-WORKTREE>` over the Netbird mesh — no host ports.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gobifrost/bifrost](https://github.com/gobifrost/bifrost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
