---
trigger: always_on
description: **All GitHub-facing text must be written in English** — PR titles, PR descriptions,
---

# Project conventions for Claude Code

## Language

**All GitHub-facing text must be written in English** — PR titles, PR descriptions,
commit messages, issue text, and code review comments. This repository's code,
comments, and existing history are all English, and PRs are read by contributors
who do not read Chinese.

**Everything under `docs/` must be written in English**, without exception —
design docs, plans, vendor integration notes, analysis write-ups. A doc drafted in
Chinese during a working session must be translated before it lands. The one
allowance is an explicitly localized top-level README (`README_zh.md`), which
exists as a translation of the English `README.md`.

Chat replies to the user in this session stay in whatever language the user is
using (usually Chinese). The rule is about what gets committed and published, not
about how we talk here.

If a PR description has already been opened in the wrong language, fix it with
`gh api -X PATCH repos/{owner}/{repo}/pulls/{n} -f body=@file` rather than
leaving it and noting the problem.

## Simplified workflow for routine tasks

**For routine development tasks, skip the full superpowers design workflow and implement
directly.** Only invoke brainstorming/writing-plans for tasks that genuinely require
upfront design:

- Complex feature additions with multiple valid architectural approaches
- Performance optimization requiring measurement and trade-off analysis
- Large-scale refactoring affecting many files or core abstractions
- Tasks where the user explicitly requests a design discussion

**Implement directly for:**
- Bug fixes with clear root cause
- Straightforward feature additions with obvious implementation
- Code cleanup and file reorganization
- Documentation updates
- Test additions
- Dependency updates

When implementing directly, still follow investigation-before-action (read relevant code
first) and verification-before-completion (run tests, check the build). The difference is
skipping the separate design phase when the path forward is clear.

## GitHub Templates and Issue/PR Guidelines

**When creating issues or pull requests, use the appropriate templates:**

### For AI Agents (Claude Code, Codex, etc.)

**READ THESE FIRST before filing any issue or PR:**
1. `.github/AI_AGENT_GUIDE.md` - Complete guidelines for AI agents
2. `.github/CLAUDE_CODE_GUIDE.md` - Claude Code specific instructions

**Required templates:**
- **Issues**: Use `.github/ISSUE_TEMPLATE/ai_agent_issue.md`
- **Pull Requests**: Use `.github/PULL_REQUEST_TEMPLATE/ai_agent_pr.md`

**Pre-submission validation:**
```bash
# Run this before creating any PR
python scripts/validate_ai_pr.py --pr-body pr_description.md
```

**Mandatory requirements for AI PRs:**
- Root cause analysis (not just symptoms)
- Investigation process documented
- Actual test output pasted (not claims like "tests pass")
- Linting results shown (`ruff check`, `ruff format --check`)
- Edge cases identified and tested
- Human reviewer assigned
- Everything in English (enforced by validation script)

**Creating an AI PR:**
```bash
# After implementation and validation
gh pr create \
  --title "fix: <description>" \
  --body-file pr_description.md \
  --label "ai-generated"
```

### For Human Contributors

Use standard templates:
- Bug reports: `.github/ISSUE_TEMPLATE/bug_report.md`
- Feature requests: `.github/ISSUE_TEMPLATE/feature_request.md`
- Platform support: `.github/ISSUE_TEMPLATE/platform_support.md`
- Operator requests: `.github/ISSUE_TEMPLATE/operator_support.md`
- Pull requests: `.github/PULL_REQUEST_TEMPLATE.md`

See `CONTRIBUTING.md` for detailed contribution guidelines.

### Strict upstream branch policy

**Never create, push, update, or delete a feature branch on an upstream repository.**
Upstream remotes are read-only for feature development, including when the current
account happens to have write access.

Before pushing any feature branch:

1. Inspect `git remote -v` and identify the contributor fork and the upstream repository.
2. Push the feature branch only to the contributor fork (normally `origin`):
   ```bash
   git push origin <feature-branch>
   ```
3. Create the pull request from `<fork-owner>:<feature-branch>` into the upstream
   repository's `main` branch:
   ```bash
   gh pr create \
     --repo <upstream-owner>/<upstream-repo> \
     --head <fork-owner>:<feature-branch> \
     --base main
   ```
4. If a feature branch was accidentally created on the upstream repository, stop,
   report it, and clean it up only as part of an explicitly authorized correction.

Do **not** run `git push <upstream> <feature-branch>` for ordinary development. A
successful push is not evidence that the push was appropriate.

## Commit Message Format

All commits should follow this format:
```
<type>: <subject>

<body>

Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>
```

**Valid types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `perf:` - Performance improvement
- `refactor:` - Code refactoring (no behavior change)
- `docs:` - Documentation changes
- `test:` - Test additions or modifications
- `ci:` - CI/CD changes
- `build:` - Build system changes

**Example:**
```
fix: resolve CUDA stream synchronization race in profiler


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flagos-ai/Torch-FL](https://github.com/flagos-ai/Torch-FL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
