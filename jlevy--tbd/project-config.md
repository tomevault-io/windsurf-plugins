---
trigger: always_on
description: IMPORTANT: You MUST read ./docs/development.md and ./docs/docs-overview.md for project
---

IMPORTANT: You MUST read ./docs/development.md and ./docs/docs-overview.md for project
documentation. (This project uses Speculate project structure.)

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below.
Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**

- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say “ready to push when you are” - YOU must push
- If push fails, resolve and retry until it succeeds

## <!-- BEGIN TBD INTEGRATION -->

* * *
## title: tbd Workflow
description: Full tbd workflow guide for agents

**`tbd` helps humans and agents ship code with greater speed, quality, and discipline.**

1. **Beads**: Git-native issue tracking (tasks, bugs, features).
   Never lose work across sessions.
   Drop-in replacement for `bd`.
2. **Spec-Driven Workflows**: Plan features → break into beads → implement
   systematically.
3. **Knowledge Injection**: 17+ engineering guidelines (TypeScript, Python, TDD,
   testing, Convex, monorepos) available on demand.
4. **Shortcuts**: Reusable instruction templates for common workflows (code review,
   commits, PRs, cleanup, handoffs).

## Installation

```bash
npm install -g get-tbd@latest
tbd setup --auto --prefix=<name>   # Fresh project (--prefix is REQUIRED: 2-8 alphabetic chars recommended. ALWAYS ASK THE USER FOR THE PREFIX; do not guess it)
tbd setup --auto                   # Existing tbd project (prefix already set)
tbd setup --from-beads             # Migration from .beads/ if `bd` has been used
```

## Routine Commands

```bash
tbd --help    # Command reference
tbd status    # Status
tbd doctor    # If there are problems

tbd setup --auto   # Run any time to refresh setup
tbd prime      # Restore full context on tbd after compaction
```

## CRITICAL: You Operate tbd — The User Doesn’t

**You are the tbd operator:** Users talk naturally; you translate their requests to tbd
actions. DO NOT tell users to run tbd commands.
That’s your job.

- **WRONG**: “Run `tbd create` to track this bug”

- **RIGHT**: *(you run `tbd create` yourself and tell the user it’s tracked)*

**Welcoming a user:** When users ask “what is tbd?”
or want help → run `tbd shortcut welcome-user`

## User Request → Agent Action

| User Says | You (the Agent) Run |
| --- | --- |
| **Issues/Beads** |  |
| “There’s a bug where …” | `tbd create "..." --type=bug` |
| “Create a task/feature for …” | `tbd create "..." --type=task` or `--type=feature` |
| “Let’s work on issues/beads” | `tbd ready` |
| “Show me issue X” | `tbd show <id>` |
| “Close this issue” | `tbd close <id>` |
| “Search issues for X” | `tbd search "X"` |
| “Add label X to issue” | `tbd label add <id> <label>` |
| “What issues are stale?” | `tbd stale` |
| **Planning & Specs** |  |
| “Plan a new feature” / “Create a spec” | `tbd shortcut new-plan-spec` |
| “Break spec into beads” | `tbd shortcut plan-implementation-with-beads` |
| “Implement these beads” | `tbd shortcut implement-beads` |
| **Code Review & Commits** |  |
| “Review this code” / “Code review” | `tbd shortcut review-code` |
| “Review this PR” | `tbd shortcut review-github-pr` |
| “Commit this” / “Use the commit shortcut” | `tbd shortcut code-review-and-commit` |
| “Create a PR” / “File a PR” | `tbd shortcut create-or-update-pr-simple` |
| “Merge main into my branch” | `tbd shortcut merge-upstream` |
| **Guidelines & Knowledge** |  |
| “Use TypeScript best practices” | `tbd guidelines typescript-rules` |
| “Use Python best practices” | `tbd guidelines python-rules` |
| “Build a TypeScript CLI” | `tbd guidelines typescript-cli-tool-rules` |
| “Improve monorepo setup” | `tbd guidelines pnpm-monorepo-patterns` or `bun-monorepo-patterns` |
| “Add golden/e2e testing” | `tbd guidelines golden-testing-guidelines` |
| “Use TDD” / “Test-driven development” | `tbd guidelines general-tdd-guidelines` |
| “Convex best practices” | `tbd guidelines convex-rules` |
| **Documentation** |  |
| “Research this topic” | `tbd shortcut new-research-brief` |
| “Document architecture” | `tbd shortcut new-architecture-doc` |
| **Cleanup & Maintenance** |  |
| “Clean up this code” / “Remove dead code” | `tbd shortcut code-cleanup-all` |
| “Fix repository problems” | `tbd doctor --fix` |
| **Sessions & Handoffs** |  |
| “Hand off to another agent” | `tbd shortcut agent-handoff` |
| “Check out this library’s source” | `tbd shortcut checkout-third-party-repo` |
| *(your choice whenever appropriate)* | `tbd list`, `tbd dep add`, `tbd close`, `tbd sync`, etc. |

**Note:** Never gitignore `.tbd/workspaces/` — the outbox must be committed to your

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jlevy/tbd](https://github.com/jlevy/tbd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
