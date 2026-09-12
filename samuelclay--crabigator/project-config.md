---
trigger: always_on
description: Guidance for coding agents (Claude Code, Codex CLI) working in this repository. `CLAUDE.md` is a symlink to this file, so both assistants read the same instructions.
---

# AGENTS.md

Guidance for coding agents (Claude Code, Codex CLI) working in this repository. `CLAUDE.md` is a symlink to this file, so both assistants read the same instructions.

## What This Project Is

Crabigator is a Rust TUI wrapper around the Claude Code, Codex, opencode, and Grok CLIs. It spawns the assistant CLI in a PTY (pseudo-terminal) and adds status widgets below the interface showing git status, file changes, session statistics, per-turn recaps, and tracked PRs. Sessions can stream to the official dashboard or a compatible self-hosted Cloudflare Worker in `workers/crabigator-api/`.

Platform selection:

```bash
crabigator                  # Uses default platform (config/env/claude)
crabigator codex            # Use Codex CLI
crabigator claude           # Use Claude Code
crabigator opencode         # Use opencode
crabigator grok             # Use Grok Build (aliases: grok-build, xai)
crabigator --platform grok  # Explicit flag
```

Other subcommands: `inspect` (view running instances), `prs` (live cross-session PR board; `--once` prints one frame — only sessions with a live mirror under /tmp appear, the durable history lives on the web dashboard's PR board; `/` search also greps each live session's transcript and shows the matched excerpt inline, Tab toggles surrounding context; `↑↓` select a session and Enter toggles a quick look pane that mirrors its live screen in the bottom half — while open, `↑↓` scroll that session's transcript and `←→` switch sessions; the default PR view shows one block per primary PR — its GitHub status inline and every touching session as a sub-row beneath it, sorted by the freshest of session activity and PR events, GitHub's updatedAt included; `p` flips to session view, the transpose: one block per session — so blocks map one-to-one to your sessions — with every PR the session touches as a sub-row beneath it; `w` watches any PR by URL or owner/repo#number — watched PRs live in the cloud like dispositions, show on every board (the open board runs `gh` for them and relays the stats), and can also be added by typing "track PR <url>" in a session; dispositions (promote/demote/dismiss) are scoped: the ✕ on a session's own strip or on a session-view sub-row applies only to that session — or to its worktree directory when the session runs in a linked worktree, so future sessions there inherit it — while the ✕ on a PR-view block or a watched PR still removes the PR group-wide; `r` shows or hides recaps, `a` cycles activity ages, and `s` toggles live/all sessions), `pair` (dashboard auth code), `recap` (enable/disable/status for turn recaps), `key` (save an Anthropic API key for recaps), `install-launcher` (macOS crabigator:// URL handler), `resume`/`continue`.

Preferences live in `~/.crabigator/config.toml` (default platform, `ide` for clickable file links, terminal emulator override, recap settings, and `[pr_board]` view preferences saved by the PR board).

## Ask Questions Liberally

**Use the AskUserQuestion tool frequently throughout development - not just during planning.**

Asking questions is encouraged and appreciated because it:
- Helps both of us think through problems more clearly
- Surfaces edge cases and requirements that might be missed
- Leads to better solutions through collaborative dialogue
- Catches misunderstandings early before code is written

Ask about:
- Clarifying requirements and desired behavior
- UI/UX preferences and design decisions
- Trade-offs between different approaches
- Edge cases and error handling
- Whether a proposed solution matches expectations
- Anything you're uncertain about

Don't assume - ask. Multiple rounds of questions are better than one large batch. Even mid-implementation, if something feels unclear or you're choosing between options, ask. The interactive back-and-forth is valuable.

## Plain Language: ISO 24495-1

All prose you write for humans must follow the ISO 24495-1 plain language principles. This applies to everything, with no exceptions: summaries of your work, PR descriptions, **git commit messages**, and **everything in a release** — release notes, highlights, and the GitHub release description:

1. **Relevant**: Include what the reader needs and nothing else. A commit message explains what changed and why it matters; it does not narrate the debugging journey or list every file touched.
2. **Findable**: Put the most important information first. The commit subject line carries the change; the body adds the "why". Summaries lead with the outcome, then supporting detail.
3. **Understandable**: Use familiar words, active voice, and short sentences. One idea per sentence. Prefer "Show the session ID in the status bar" over "Implement session identifier visibility enhancements".
4. **Usable**: Write so the reader can act on it. A summary should let the user verify the work; a commit message should let a future reader decide whether this commit is the one they're hunting for.

For commit messages specifically:
- Subject line: imperative mood, plain words, describes the user-visible or behavioral change (matches existing history, e.g. "Keep previous recap visible between turns").

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samuelclay/crabigator](https://github.com/samuelclay/crabigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
