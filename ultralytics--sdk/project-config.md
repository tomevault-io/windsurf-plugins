---
trigger: always_on
description: Repository guidance for coding agents. `CLAUDE.md` is a symlink to this file. This repository generates and publishes the `ultralytics-platform` Python SDK and its `ul` CLI (Python>=3.11) from the Platform OpenAPI contract.
---

# AGENTS.md

Repository guidance for coding agents. `CLAUDE.md` is a symlink to this file. This repository generates and publishes the `ultralytics-platform` Python SDK and its `ul` CLI (Python>=3.11) from the Platform OpenAPI contract.

## Core Principles (CRITICAL)

**Less is more. The simplest solution is the best solution.** The action hierarchy for every change: **Delete > Replace > Add**.

1. **Solve at the owner**: Put behavior in the code path that owns or observes it. For fixes, never guard a symptom with a staleness check, initialization flag, skip-first-call branch, or `try/except` around broken logic; relocate the trigger and delete the wrong path. For features, extend the existing owner rather than creating a parallel abstraction.
2. **Search and reuse first**: Search the whole repository before creating a feature, component, helper, workflow, or utility. Reuse or adapt what exists, consolidate in-scope duplication in the shared owner, and delete duplicate paths. Three similar lines beat a helper nobody else calls.
3. **Delete and modify existing code before creating new code**: Bugfixes are net-negative by default unless deletion and relocation are demonstrably impossible. A new file must first prove it cannot fit cleanly in an existing owner.
4. **Keep scope minimal**: Implement only the simplest complete solution. Avoid impossible-state handling, speculative flags, compatibility shims, policy scaffolding, and unrelated cleanup. Tests are out of scope by default — rely on existing coverage and focused validation; only an uncovered, high-risk regression path justifies minimal new test code.
5. **Ship zero-regression, production-ready changes**: Understand what you remove instead of retaining broken code as insurance. Remove unused imports, functions, types, files, and comments; run relevant cleanup checks; and thoroughly debug and validate the changed owner. Do not break existing features or workflows unless the PR intentionally removes them with evidence.

**Review gate:** for every addition, the reviewer decides whether deleting or changing existing code would have fixed the problem instead — if it would, that is a blocking finding. A missing or thin PR description is never itself a finding.

NEVER push to `main`. NEVER force push. Always start work in a new git worktree (`git worktree add`) on a feature branch and open a PR — never edit the primary checkout directly, it may hold in-flight work.

## PR Workflow

After opening a PR:

1. Wait for the automated PR review and any header or spelling commits from Ultralytics Actions (`format.yml`), then pull and address every finding.
2. Review the full diff in-session against the Core Principles, performance, and the review gate above, then batch the fixes into one commit and push. After each round of bot or human commits, pull and resume the same reviewer on `<last-reviewed-sha>..HEAD` plus anything that delta could have invalidated. Repeat until the local head matches the live head.
3. Hand off or merge only on a clean final pass: one cold full-diff review returning LGTM with no findings, on a head that is still live at merge time.
4. Never fight other commits: Ultralytics Actions pushes header and spelling commits, and multiple users may work on the same PR. `git pull --rebase` before pushing; never reset or revert commits you did not author.
5. After the PR merges, clean up: remove local worktrees and branches for it, then `git checkout main && git pull`.

## API and SDK versioning (CRITICAL)

**One version, one owner: the upstream Platform API contract's `info.version`. The Python SDK package version MUST equal the API contract version it contains.** This repository consumes the deployed contract through automation; it does not choose release versions.

- Keep public documentation and PR text scoped to the public upstream API contract. Do not include private repository names, internal paths, or private PR links.
- NEVER set `python.version` in `openapi.config.json`, independently bump the SDK patch, or edit versions in generated files. Do not restore automatic patch bumps or `max(API version, SDK version)` logic.
- This also applies to SDK-only CLI/help/auth fixes and generator improvements. Merge the source fix, coordinate a contract version bump and deployment with the Platform API maintainers, then let SDK contract synchronization regenerate and publish that same version. Do not manually repair the snapshot or generated descendants to manufacture a release.
- Before updating a consumer's minimum SDK requirement, verify the published wheel contains the required behavior and its version matches the deployed API. A successful install or green CI alone does not prove this.
- If SDK versions have already been published ahead of the API, the API maintainers must advance the upstream contract beyond every published SDK version, then synchronize. Never downgrade the SDK, reuse a published version, or claim the offset will self-heal. API `0.1.50` with SDK `0.1.52` is INVALID; both at `0.1.52` is valid.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ultralytics/sdk](https://github.com/ultralytics/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
