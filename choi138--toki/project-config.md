---
trigger: always_on
description: This file is the high-level policy entry point for AI tooling in this repository.
---

# Workspace Rules

This file is the high-level policy entry point for AI tooling in this repository.
Keep detailed source rules out of `AGENTS.md`; store them in the Codex conventions
files referenced below.

## Source Of Truth

| Topic | Source | When |
| --- | --- | --- |
| Toki Source Conventions | `/project-conventions` skill or `.agents/skills/project-conventions/conventions.md` plus task-specific files under `.agents/skills/project-conventions/references/` | Before editing `menubar/Toki/**/*.swift`, `menubar/TokiTests/**/*.swift`, `menubar/project.yml`, `.swiftformat`, `.swiftlint.yml`, or app resources |
| Local Codex Review Loop | `/codex-review-loop` skill or `.agents/skills/codex-review-loop/SKILL.md` | Local diff review, approved finding fixes, and bounded re-review |
| Git Workflow | `.agents/conventions/git-workflow.md` | Branch / commit / PR work |

The shared `.agents` hooks provide soft enforcement for `/project-conventions`
at session start, on matching prompts, and on Bash commands that appear to
modify Toki source, tests, resources, or XcodeGen configuration.
This is a Codex-native reminder layer, not a hard block for non-shell edits.

## Toki Workflow

Toki is a macOS menu bar app built with Swift, SwiftUI, XcodeGen, SwiftFormat,
and SwiftLint.

1. Inspect the existing local pattern before changing code.
2. Keep app entry code in `menubar/Toki/App`, domain models and builders in
   `menubar/Toki/Domain`, platform and data readers in `menubar/Toki/Infrastructure`,
   feature UI/view-model code in `menubar/Toki/Features`, and tests in
   `menubar/TokiTests`.
3. Prefer editing `menubar/project.yml` and regenerating with `xcodegen generate`
   from `menubar/` for project configuration changes. Avoid hand-editing
   `menubar/Toki.xcodeproj` unless
   the user explicitly asks for a project-file-only fix or regeneration is not
   possible.
4. Treat local agent logs, usage databases, security audit findings, and secret
   detection data as sensitive. Do not add telemetry, network transmission, or
   unmasked logging for those values unless explicitly requested.
5. Keep SwiftUI views focused on presentation. Move parsing, aggregation,
   pricing, security scanning, persistence, and time calculations into domain,
   infrastructure, or view-model code.
6. Add or update focused tests for reader behavior, usage aggregation,
   formatting, security audit behavior, settings, and view-model logic when
   those surfaces change.

## Required Checks

Run the smallest relevant subset while iterating. Before a PR or broad source
change, prefer the full set:

```bash
swiftformat . --lint
swiftlint lint --strict --quiet
cd menubar
xcodegen generate
xcodebuild test \
  -project Toki.xcodeproj \
  -scheme Toki \
  -destination "platform=macOS" \
  CODE_SIGN_IDENTITY="" \
  CODE_SIGNING_REQUIRED=NO \
  CODE_SIGNING_ALLOWED=NO
```

## Global Safety Rules

These rules apply to every repository and every session.

### Remote Branch And Pull Request Safety

- Treat any remote branch deletion as a destructive operation. This includes
  `git push origin --delete <branch>`, `git push <remote> --delete <branch>`,
  `git push origin :<branch>`, and equivalent GitHub API or UI actions.
- Never delete remote branches unless the user explicitly asks to delete
  remote/origin branches by name. Phrases like "main only", "clean branches",
  or "remove old branches" mean local branch cleanup only.
- For branch cleanup, default to local branches plus `git fetch --prune`; do
  not delete branches from `origin` or any other remote.
- Before deleting any remote branch, inspect whether it backs an open pull
  request with:

  ```bash
  gh pr list --head <branch> --state open --json number,title,author,url,headRefName,baseRefName
  ```

- If any open pull request exists for the branch, do not delete the branch
  unless the user explicitly confirms the exact repository, branch name, and PR
  number.
- If the pull request author is not the current authenticated GitHub user, stop
  the current task immediately. Do not continue with the action in the same
  flow. Tell the user that the action would affect another person's PR, include
  the repo/branch/PR/author, and ask for explicit confirmation in a new user
  response before doing anything else.
- After asking for confirmation about another person's PR, end the current turn
  and do not run any further tools or commands until the user replies.
- Do not use `--no-verify` to bypass hooks for remote branch deletion unless
  the user explicitly requested the exact remote branch deletion after being
  told which PRs or users may be affected.
- Never close, merge, mark ready/draft, retarget, force-push, delete the head
  branch of, resolve review threads on, or otherwise mutate another person's
  pull request without stopping and asking the user again first.
- Reading another person's PR is allowed. Any write or destructive action
  against another person's PR, branch, review, or comments requires a fresh
  explicit confirmation from the user after you have identified the affected PR.

## CodeGraph

This project has a CodeGraph MCP server (`codegraph_*` tools) configured.
CodeGraph is a tree-sitter-parsed knowledge graph of symbols, edges, and files.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [choi138/toki](https://github.com/choi138/toki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
