---
trigger: always_on
description: Shared guardrails distilled from the various `~/Projects/*/AGENTS.md` files (state as of **November 15, 2025**). This document highlights the rules that show up again and again; still read the repo-local instructions before making changes.
---

<shared>
# AGENTS.md

Shared guardrails distilled from the various `~/Projects/*/AGENTS.md` files (state as of **November 15, 2025**). This document highlights the rules that show up again and again; still read the repo-local instructions before making changes.

Usage: In repo copies, the shared content lives inside `<shared>…</shared>` and the tool list inside `<tools>…</tools>`. Keep those tagged blocks identical across repos; anything outside them is repo-local and can be customized freely.

## Codex Global Instructions

- Keep the system-wide Codex guidance at `~/.codex/AGENTS.md` (the Codex home; override via `CODEX_HOME` if needed) so every task inherits these rules by default.

## General Guardrails

### Intake & Scoping

- Open the local agent instructions plus any `docs:list` summaries at the start of every session. Re-run those helpers whenever you suspect the docs may have changed.
- Review any referenced tmux panes, CI logs, or failing command transcripts so you understand the most recent context before writing code.

### Tooling & Command Wrappers

- Use the command wrappers provided by the workspace (`./runner …`, `scripts/committer`, `pnpm mcp:*`, etc.). Skip them only for trivial read-only shell commands if that’s explicitly allowed.
- Stick to the package manager and runtime mandated by the repo (pnpm-only, bun-only, swift-only, go-only, etc.). Never swap in alternatives without approval.
- When editing shared guardrail scripts (runners, committer helpers, browser tools, etc.), mirror the same change back into the `agent-scripts` folder so the canonical copy stays current.
- Ask the user before adding dependencies, changing build tooling, or altering project-wide configuration.
- When discussing dependencies, always provide a GitHub URL.
- Keep the project’s `AGENTS.md` `<tools></tools>` block in sync with the full tool list from `TOOLS.md` so downstream repos get the latest tool descriptions.

### tmux & Long Tasks

- Run any command that could hang (tests, servers, log streams, browser automation) inside tmux using the repository’s preferred entry point.
- Do not wrap tmux commands in infinite polling loops. Run the job, sleep briefly (≤30 s), capture output, and surface status at least once per minute.
- Document which sessions you create and clean them up when they are no longer needed unless the workflow explicitly calls for persistent watchers.

### Build, Test & Verification

- Before handing off work, run the full “green gate” for that repo (lint, type-check, tests, doc scripts, etc.). Follow the same command set humans run—no ad-hoc shortcuts.
- Leave existing watchers running unless the owner tells you to stop them; keep their tmux panes healthy if you started them.
- Treat every bug fix as a chance to add or extend automated tests that prove the behavior.
- When someone asks to “fix CI,” use the GitHub CLI (`gh`) to inspect, rerun, and unblock failing workflows on GitHub until they are green.

### Code Quality & Naming

- Refactor in place. Never create duplicate files with suffixes such as “V2”, “New”, or “Fixed”; update the canonical file and remove obsolete paths entirely.
- Favor strict typing: avoid `any`, untyped dictionaries, or generic type erasure unless absolutely required. Prefer concrete structs/enums and mark public concurrency surfaces appropriately.
- Keep files at a manageable size. When a file grows unwieldy, extract helpers or new modules instead of letting it bloat.
- Match the repo’s established style (commit conventions, formatting tools, component patterns, etc.) by studying existing code before introducing new patterns.

### Git, Commits & Releases

- Invoke git through the provided wrappers, especially for status, diffs, and commits. Only commit or push when the user asks you to do so.
- To resolve a rebase, `git add`/`git commit` is allowed.
- Follow the documented release or deployment checklists instead of inventing new steps.
- Do not delete or rename unfamiliar files without double-checking with the user or the repo instructions.

### Documentation & Knowledge Capture

- Update existing docs whenever your change affects them, including front-matter metadata if the repo’s `docs:list` tooling depends on it.
- Whenever doing a large refactor, track work in `docs/refactor/<title><date>.md`, update it as you go, and delete it when the work is finished.
- Only create new documentation when the user or local instructions explicitly request it; otherwise, edit the canonical file in place.
- When you uncover a reproducible tooling or CI issue, record the repro steps and workaround in the designated troubleshooting doc for that repo.
- Routine test additions don’t require changelog entries; reserve changelog lines for user-visible behavior changes.

### Troubleshooting & Observability

- Design workflows so they are observable without constant babysitting: use tmux panes, CI logs, log-tail scripts, MCP/browser helpers, and similar tooling to surface progress.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [steipete/mcporter](https://github.com/steipete/mcporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
