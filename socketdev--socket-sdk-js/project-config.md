---
trigger: always_on
description: 🚨 **MANDATORY**: Act as principal-level engineer with deep expertise in TypeScript, Node.js, and SDK development.
---

# CLAUDE.md

🚨 **MANDATORY**: Act as principal-level engineer with deep expertise in TypeScript, Node.js, and SDK development.

<!-- <fleet-canonical> -->

## 📚 Fleet

- Identify users by git credentials; use their actual name and "you/your" directly; shorthand has fixed meanings ("commit as you go", "land it", "update `<socket-pkg>`" = its `-stable` alias too); a task/issue ref carries its subject on first mention — "#12 (remove npm-run-all2)", never a bare `#12`. (`.claude/hooks/fleet/reply-prose-nudge/`) [`vocabulary`](docs/agents.md/fleet/vocabulary.md)
- 🚨 Multiple Claude sessions may target one checkout — never run a git command that mutates state outside the file you just edited (no stash / blanket add / branch switch / hard reset / restore-dot / force clean in the primary checkout); branch + sub-agent work goes in a `git worktree`. (`.claude/hooks/fleet/no-revert-guard/`) [`parallel-claude-sessions`](docs/agents.md/fleet/parallel-claude-sessions.md)
- 🚨 Local main is canonical — origin ahead by own/bot squash commits ≠ newer truth; reconcile FORWARD (amend or lease-force-push), never reset/rewind local to origin. (`.claude/hooks/fleet/{unpushed-main-nudge,no-revert-guard}/`) [`parallel-claude-sessions`](docs/agents.md/fleet/parallel-claude-sessions.md)
- 🚨 Active-edits ledger coordinates concurrent actors — per-actor edit timestamps block editing a path another live actor wrote within 5 min, exempt live-foreign-owned paths, and block open-ended wait promises while a live actor is present; converge, arm a Monitor, or hand off via a `.claude/plans/` doc. (`.claude/hooks/fleet/{active-edits-ledger,live-edit-collision-guard}/`) [`parallel-claude-sessions`](docs/agents.md/fleet/parallel-claude-sessions.md)
- 🚨 Codex companion sessions (`CODEX_COMPANION_SESSION_ID`) are quick checks — blocked past a 1-min budget. Bypass: `Allow codex-long-session bypass`. (`.claude/hooks/fleet/codex-session-budget-guard/`) [`parallel-claude-sessions`](docs/agents.md/fleet/parallel-claude-sessions.md)
- Never hard-code `main` in scripts — resolve via `git symbolic-ref refs/remotes/origin/HEAD`, fall back `main` → `master`; applies to worktree creation, base-ref resolution, PR base detection, hook scripts. (`.claude/hooks/fleet/default-branch-guard/`)
- 🚨 Never write a real customer/company name, private repo, Linear ref, or Slack thread into any public/committed surface — fictional slugs only; `fleet-repos.json` is the SOLE sanctioned private-name list. (`.claude/hooks/fleet/{private-name-nudge,public-surface-nudge,no-private-path-in-source-guard,no-private-ref-in-tests-docs-guard,release-workflow-guard}/`) [`public-surface-hygiene`](docs/agents.md/fleet/public-surface-hygiene.md) [`pull-request-target`](docs/agents.md/fleet/pull-request-target.md)
- 🚨 Root `README.md` follows the fleet skeleton — 5 level-2 sections in order (a `freeform-readme` roster opt-in exempts product/marketplace repos from the sections; follow-badges + no-leak + no-sibling-path stay universal); no the fleet source repo mentions; no sibling-relative script commands. Bypass: `Allow readme-fleet-shape bypass`. (`.claude/hooks/fleet/readme-fleet-shape-guard/`) [`public-surface-hygiene`](docs/agents.md/fleet/public-surface-hygiene.md)
- 🚨 Conventional Commits `<type>(<scope>): <description>`, lowercase, NO AI attribution — in commits AND every GitHub prose surface AND external MCP surfaces (Linear, Slack); no placeholder subject; push direct → PR only on rejection. (`.claude/hooks/fleet/{commit-message-format-guard,no-github-ai-attribution-guard,no-placeholder-commit-subject-guard,commit-pr-nudge,no-non-fleet-push-guard,non-fleet-pr-issue-ask-guard}/`) [`commit-cadence-format`](docs/agents.md/fleet/commit-cadence-format.md)
- 🚨 Run human-facing prose through the `prose` skill before it lands; PR/issue/comment bodies use conversational mode; CHANGELOG = user-visible behavior only, one-line bullets; cascade + bot output exempt. (`.claude/hooks/fleet/{changelog-entry-shape-nudge,convo-prose-nudge,anti-prose-guard,prose-code-format-nudge}/`) [`prose-style-and-doctrine`](docs/agents.md/fleet/prose-style-and-doctrine.md)
- PR review comments use the fleet comment format — severity-sorted `<details>` `<abbr>` circles (🔴🟠🟡🟢), `Fix idea 💡:` labels, `item N _(title)_` refs, junior-dev complete sentences via the `prose` skill, dup-PR scan, no bot repetition; comment-only — never approve, never request-changes (`no-pr-review-verdict-guard`); validate drafts with `scripts/fleet/lint-pr-comment.mts`. [`pr-review-comments`](docs/agents.md/fleet/pr-review-comments.md)
- Some fleet repos squash the default branch on a cadence — commits are ephemeral, so land fast and don't fuss; disjoint parallel work NEVER waits for a commit boundary (`squashing-history` skill); attribution strips via `scripts/fleet/strip-ai-attribution.mts`. Bypass: `Allow squash-history-nudge bypass`. (`.claude/hooks/fleet/{squash-history-nudge,parallel-agent-on-stop-nudge,attribution-rewrite-nudge}/`) [`history-rewrites`](docs/agents.md/fleet/history-rewrites.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SocketDev/socket-sdk-js](https://github.com/SocketDev/socket-sdk-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
