---
trigger: always_on
description: **Pipeline: `test` → `dev` → `main`** (ONE DIRECTION ONLY!)
---

# Development Context - DEXBot2

## Branch Strategy
**Pipeline: `test` → `dev` → `main`** (ONE DIRECTION ONLY!)

- **test**: Primary development branch (where work happens)
- **dev**: Integration/staging (merged from test)
- **main**: Production-ready (merged from dev)

⚠️ **KEY RULE**: Always merge **test → dev**, NEVER dev → test
⚠️ **KEY RULE**: See **Absolute Git Action Gate** below for all write-action authorization rules.
⚠️ **KEY RULE**: Default to manual merge/push flow for branch promotion when requested, unless the user specifically asks to use one of the sync scripts.

## Absolute Git Action Gate (User-Directed Writes)

**Agent must NOT proactively ask for or execute git write actions.**
The agent only runs git write actions when the user clearly requests them.

Git write actions include:
- `git add`
- `git commit`
- `git commit --amend`
- `git reset` (any mode)
- `git rebase`
- `git merge`
- `git push`
- `git tag`
- `git checkout` / `git switch` to another branch

Branch-promotion scripts (use force-push, not merge):
- `npm run ptest` — push local `test` to `origin/test`
- `npm run pdev` — force-push `test` → `origin/dev` (skips merge)
- `npm run pmain` — force-push `test` → `origin/dev` **and** `origin/main` (skips dev staging and merge entirely)

Read-only git commands are always allowed (for example: `git status`, `git diff`, `git log`, `git show`).

Interpretation rules:
1. If a user clearly asks for a git write action, execute it.
2. Short approvals like "yes", "ok", "do it", or "go ahead" are valid confirmation when they clearly refer to the immediately previous proposed action.
3. If wording is ambiguous, ask one clarifying question before running destructive actions.
4. `git commit --amend` is allowed when explicitly requested by the user.
5. Before a git write action, restate the user authorization in one short line.

See `docs/WORKFLOW.md` for detailed workflow guide.

## Commit Quality Standard
When creating commits, prefer high-context commit messages for non-trivial fixes/features.

- **Subject**: concise conventional prefix (`fix:`, `feat:`, `docs:`) with clear scope.
- **Body required for substantial changes**: explain **why**, not only what changed.
- **Structure**:
  1. Short problem statement/context
  2. Per-fix sections with file path(s) and behavioral impact
  3. Risk/edge-case notes when relevant
  4. Validation/testing notes (commands or scenario checks)
- **Formatting**: use readable markdown headers/bullets in commit body for scanability.
- **CLI formatting safety**:
  - Never use `/n` or literal `\\n` text as a newline placeholder in commit/PR bodies.
  - Always pass real newlines to Git/GitHub (multi-line body), not escaped newline text.
  - Prefer heredocs for reliability when using `git commit` and `gh pr create`.
- **Atomicity**: keep unrelated edits out of the commit; document only included changes.
- **Tone**: use professional, concise language — no personal commentary, verbose explanations, or markdown section headers in commit bodies. Stick to the facts: what changed and why.
- **No personal information in commits, PRs, or docs**: never include real account names, bot names, market/pair names (e.g. live trading pairs), addresses, or other real identifiers in commit messages, PR titles/bodies, release notes, CHANGELOG, README, or any documentation. Describe incidents generically (e.g. "a live market-pair bot") and use placeholders (e.g. `1.2.x`, `account-name`, `<market-pair>`) in examples. Before committing, scan the message for identifiers that slipped in; if one lands in an existing commit, amend (or follow up) to strip it. This applies to all new content and existing entries being edited.
- **Unrelated working-tree files**: if the diff includes pre-existing dirty files the user wants in a separate commit, use `git add <scope>` to stage only the relevant files. NEVER run `git checkout -- <file>` or `git restore <file>` on those files — that destroys the working-tree content. Let them stay dirty for a future commit.

Recommended CLI patterns (newline-safe):

```bash
# Commit message with proper markdown/newlines
git commit -F- <<'EOF'
fix: <short summary>

<context>

## <Fix area>
- Problem:
- Impact:
- Solution:

## Testing Notes
- <test command>
EOF

# PR body with proper markdown/newlines
gh pr create --title "<title>" --body-file - <<'EOF'
## Summary
- <item>

## Testing
- <command>
EOF
```

## Key Files

### Entry Points
- `dexbot.ts` - Main CLI entry point
- `bot.ts` - Alternative bot starter
- `pm2.ts` - PM2 process management
- `unlock.ts` - Single-prompt starter (no PM2)
- `credential-daemon.ts` - Credential daemon

### Core Bot
- `modules/dexbot_class.ts` - Core bot class, lifecycle orchestration, and shared runtime wiring
- `modules/dexbot_fill_runtime.ts` - Fill processing runtime and replay-safe accounting helpers
- `modules/dexbot_maintenance_runtime.ts` - Maintenance runtime for sync loops and grid checks
- `modules/constants.ts` - Centralized configuration and tuning parameters
- `modules/bitshares_client.ts` - BitShares connection and node management
- `modules/node_manager.ts` - Multi-node health checking and failover
- `modules/fund_registry.ts` - Shared-account fund registry with cross-bot invariants

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [froooze/DEXBot2](https://github.com/froooze/DEXBot2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
