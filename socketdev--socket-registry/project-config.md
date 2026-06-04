---
trigger: always_on
description: **MANDATORY**: Act as principal-level engineer. Follow these guidelines exactly.
---

# CLAUDE.md

**MANDATORY**: Act as principal-level engineer. Follow these guidelines exactly.

<!-- BEGIN FLEET-CANONICAL — sync via socket-wheelhouse/scripts/sync-scaffolding.mts. Do not edit downstream. -->

## 📚 Wheelhouse Standards

### Identifying users

Identify users by git credentials and use their actual name. Use "you/your" when speaking directly; use names when referencing contributions (enforced by `.claude/hooks/fleet/voice-and-tone-reminder/`).

### Parallel Claude sessions

🚨 Multiple Claude sessions may target the same checkout (parallel agents, terminals, or worktrees on the same `.git/`). **The umbrella rule:** never run a git command that mutates state belonging to a path other than the file you just edited. Forbidden in the primary checkout: `git stash`, `git add -A` / `git add .` (enforced by `.claude/hooks/fleet/overeager-staging-guard/`; bypass: `Allow add-all bypass`), `git checkout/switch <branch>`, `git reset --hard <non-HEAD>`. Branch work goes in a `git worktree`. Cross-repo imports via `@socketsecurity/lib/...` only, never `../<sibling-repo>/...` (enforced by `.claude/hooks/fleet/cross-repo-guard/`). Dirty paths you didn't author this session + that changed recently are likely another live agent — never mutate over them; stage only your own files (enforced by `.claude/hooks/fleet/parallel-agent-on-stop-reminder/`, enforced by `.claude/hooks/fleet/parallel-agent-staging-guard/`; bypass `Allow parallel-agent-staging bypass`). Full prohibition list + worktree recipe in [`docs/claude.md/fleet/parallel-claude-sessions.md`](docs/claude.md/fleet/parallel-claude-sessions.md).

### Default branch fallback

Never hard-code `main` in scripts — a few legacy repos still use `master`. Resolve via `git symbolic-ref refs/remotes/origin/HEAD`, fall back to `main` then `master`:

```bash
BASE=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
[ -z "$BASE" ] && git show-ref --verify --quiet refs/remotes/origin/main && BASE=main
[ -z "$BASE" ] && git show-ref --verify --quiet refs/remotes/origin/master && BASE=master
BASE="${BASE:-main}"
```

Apply in: worktree creation, base-ref resolution for `git diff`/`git rev-list`, PR base detection, hook scripts walking history. Doc examples may write `main` for clarity; scripts must look up. Order matters — `main → master` matches fleet reality; reversing would mispick during rename migrations (enforced by `.claude/hooks/fleet/default-branch-guard/`).

### Public-surface hygiene

🚨 Never write a real customer / company name, private repo / internal project name, or Linear ref (`SOC-123`, `ENG-456`, Linear URLs) into a commit, PR, issue, comment, or release note. No denylist — a denylist is itself a leak (enforced by `.claude/hooks/fleet/{private-name-guard,public-surface-reminder}/`).

🚨 Never `gh workflow run|dispatch` against publish / release / build-release workflows (enforced by `.claude/hooks/fleet/release-workflow-guard/`). Bypass: `gh workflow run -f dry-run=true` (workflow declares `dry-run:` input) OR `Allow workflow-dispatch bypass: <workflow>` typed verbatim. `workflow_dispatch.inputs` keys are kebab-case.

🚨 **Workflow YAML invariants:** SHA-pinned `uses:` lines need a `# <tag> (YYYY-MM-DD)` comment; `run:` blocks with multi-line `gh ... --body "..."` break YAML — always `--body-file <path>`; `pull_request_target` is privileged and never combines with fork-head checkout + execute. External-issue refs (`<owner>/<repo>#<num>`) in commits / PR bodies spam upstream maintainers — only `SocketDev/<repo>#<num>` is allowed inline; link upstream refs in PR _description prose_ instead. Bypass: `Allow external-issue-ref bypass`.

Full ruleset + threat model + bypass surface in [`docs/claude.md/fleet/public-surface-hygiene.md`](docs/claude.md/fleet/public-surface-hygiene.md) and [`docs/claude.md/fleet/pull-request-target.md`](docs/claude.md/fleet/pull-request-target.md).

### Canonical README

🚨 Root `README.md` follows the fleet skeleton — 5 level-2 sections in order (Why this repo exists / Install / Usage / Development / License), no `socket-wheelhouse` mentions (it's a private repo), no sibling-relative script commands (e.g. `node ../socket-foo/scripts/...` fails for outside readers). Canonical skeleton: `socket-wheelhouse/template/README.md`. Bypass: `Allow readme-fleet-shape bypass` (enforced by `.claude/hooks/fleet/readme-fleet-shape-guard/`).

### Commits & PRs

🚨 Conventional Commits `<type>(<scope>): <description>`, lowercase type, NO AI attribution (enforced by `.claude/hooks/fleet/commit-message-format-guard/`, enforced by `.claude/hooks/fleet/commit-pr-reminder/`; bypasses `Allow commit-format bypass` / `Allow ai-attribution bypass`). Push direct → PR only on rejection. NEVER push, open PRs, file issues, or create releases against a non-fleet repo without confirmation (bypasses `Allow non-fleet-push bypass` / `Allow non-fleet-publish bypass`; enforced by `.claude/hooks/fleet/no-non-fleet-push-guard/`, enforced by `.claude/hooks/fleet/non-fleet-pr-issue-ask-guard/`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SocketDev/socket-registry](https://github.com/SocketDev/socket-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
