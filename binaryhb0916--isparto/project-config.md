---
trigger: always_on
description: iSparto is an AI Agent Team workflow framework that turns single-agent Claude Code into a team with distinct roles (Lead + Teammate + Developer + Doc Engineer + Process Observer). Target users are independent developers. Current stage: open-source core workflow released, dogfooding in progress.
---

# iSparto

## Project Overview
iSparto is an AI Agent Team workflow framework that turns single-agent Claude Code into a team with distinct roles (Lead + Teammate + Developer + Doc Engineer + Process Observer). Target users are independent developers. Current stage: open-source core workflow released, dogfooding in progress.

## Tech Stack
- Language: Shell (Bash), Markdown
- Framework: None (pure configuration project, driven by Claude Code slash commands + MCP)
- Platform: macOS (iTerm2 + tmux 3.x — tmux required since v0.8.0 for Independent Reviewer's `codex exec` invocation in a tmux pane)
- Build: No build step
- Other: Codex MCP Server (npx codex-mcp-server)

## Development Rules
- Communicate and generate documentation in the user's language (English or Chinese only)
- Any code/command change must synchronously update the corresponding documentation (README, docs/, command header comments)
- Product direction changes must be written into documentation, not just discussed in conversation
- Ask me first about uncertain product questions; do not decide on your own
- **plan.md update cadence:** update `docs/plan.md` either per-task (in the same commit as the task work) OR per-Wave (in the T10/close-out commit that lists all task completions with commit hashes — the Wave-close approach is acceptable when the Wave runs as a single atomic work session on a dedicated branch). Wave-completion entries and cross-session BLOCKING markers are written by `/end-working` as part of the commit it generates, because that is the step that knows the Wave is fully complete. If a fix session does not correspond to any plan.md entry (e.g., a bug fix not tied to any Wave), no plan.md update is required.
- **plan.md verification-count accuracy:** when a Wave completion entry records a commit count, compute it mechanically via `git log --oneline --no-merges <wave-base>..HEAD | wc -l` — not by estimation. For entries authored pre-commit (standard `/end-working` cadence, where the Wave entry ships inside the same commit it documents), write the projected count and re-verify via the same command immediately after the commit lands; if mismatch, amend before push. Applies to every Wave close-out. The `<wave-base>` is the commit where the Wave branch diverged from the target branch.
- **Single TODO source (plan.md Backlog is authoritative):** All TODOs, deferred actions, framework rule-fix candidates, and open commitments — including Process Observer audit findings about the framework itself, user-direction commitments surfaced in conversation, and Wave-internal deferrals — must be written to `docs/plan.md`'s Backlog section. Writing new backlog-type content to any other file is a workflow violation. Specifically forbidden channels: (a) creating `docs/framework-feedback-*.md` files (pattern retired 2026-04-17); (b) embedding deferred items in Wave-entry prose (e.g., "Out of scope" paragraphs) without a parallel Backlog row; (c) storing project-level work items in `memory/` — memory is for awareness information (user preferences, project facts, external references) only, not work; (d) listing "next-session to-dos" only in `docs/session-log.md` Notes without a Backlog row. `/end-working` performs a TODO-homing audit that blocks the commit if any new backlog-type content lands outside `docs/plan.md`.
- Do not develop directly on main; feat/ for new features, fix/ for bug fixes, hotfix/ for urgent fixes, docs/ for pure documentation commits, release/ for releases
- install.sh changes must remain backward compatible (existing users must still be able to uninstall)
- Changes to command templates (commands/*.md) must be verified not to break existing users' /migrate and /init-project flows
- After completing all reviews, automatically create PR and merge to main — no manual user review needed
- Releases must use the `/release` command — manual `git tag`, `git push origin <tag>`, or any operation on main is not allowed. The release flow is fully automated by `scripts/release.sh`
- This project is the framework itself; all Tier 1 System Prompt Layer files (as defined in Documentation Language Convention) fall within the self-referential boundary — Lead edits directly, and Process Observer interceptions can be approved. This includes both subdirectory files (`commands/`, `templates/`, `scripts/`, `hooks/`, `agents/`, `docs/`, `lib/`) and root-level files (`CLAUDE.md`, `CLAUDE-TEMPLATE.md`, `bootstrap.sh`, `install.sh`, `isparto.sh`). Tier 2/3/4 documentation (other `docs/*.md`, `README*.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, `VERSION`) is also in scope for direct Lead edits under the same framework self-referential principle.

## Documentation Language Convention

iSparto adopts a four-tier language architecture for all documentation and system prompts:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BinaryHB0916/iSparto](https://github.com/BinaryHB0916/iSparto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
