---
trigger: always_on
description: Perform thorough code reviews for Swift/SwiftUI code, including spec adherence (PR description + linked issues), code quality, architecture, performance, security, Swift 6+ best practices, project standards from .claude/CLAUDE.md, and meta-feedback on recurring patterns that suggest gaps in the agent's instructions. Use when reviewing PRs/MRs (especially AI-generated ones), performing quality audits, validating against original spec, or providing structured feedback with severity levels and impr
---


# Swift/SwiftUI Code Review Skill

Multi-layer review covering Swift 6+ concurrency, SwiftUI patterns, performance, security, architecture, and project-specific standards. Reads `.claude/CLAUDE.md` and outputs Critical/High/Medium/Low severity findings with `file:line` references and before/after code examples.

## When to Use This Skill

- "Review this PR"
- "Review my code" / "Review my changes" / "Review uncommitted changes"
- "Code review for [component]"
- "Audit this codebase" / "Check code quality"
- "Review against .claude/CLAUDE.md" / "Check if this follows our coding standards"
- "Architecture review" / "Performance audit" / "Security review"
- "Review this PR against the spec"
- "Did the agent miss anything from issue #123?"
- "What rules am I missing in CLAUDE.md based on this PR?"
- "Review this AI-generated PR"

## Workflow

### Phase 0 — Resolve Scope

**Objective**: Produce a canonical scope object before any analysis begins. Every later phase reads from this object — no phase fetches the changeset independently.

#### Mode Detection

Run these checks in order; stop at the first match:

| Priority | Condition | Mode |
|----------|-----------|------|
| 1 | PR number or URL supplied by user | **PR** — `gh pr view <n> --json files,baseRefName` |
| 2 | Explicit file paths supplied by user | **File** — supplied paths → `scope.modified`, skip detection |
| 3 | `gh pr view --json number` returns a result for current branch | **PR** (auto-detected) |
| 4 | "staged" or "cached" in user's invocation | **Staged** — `git diff --cached --name-status` |
| 5 | Default | **Local** — `git diff --name-status <base>...HEAD` + `git diff --name-status` (union) |

**Base branch resolution** (for PR auto-detect and Local mode):

```bash
# 1. PR metadata (authoritative when a PR exists)
gh pr view --json baseRefName --jq '.baseRefName'
# 2. Remote default branch
git rev-parse --abbrev-ref origin/HEAD
# 3. Last resort
git branch -r | grep -E 'origin/(main|master)$' | head -1
```

If `gh` is unavailable or unauthenticated, announce loudly and fall back to Local:
> "`gh` unavailable — falling back to local mode against `origin/main`. Run `gh auth login` for full PR scope detection."

#### Scope Object

```
scope = {
  modified:         Set<Path>   // first-class findings — full file, all severity levels
  deleted:          Set<Path>   // spec-adherence reasoning only — skip per-file analysis loop
  testsForModified: Set<Path>   // coverage findings → main report; other findings → Adjacent Observations
  related:          Set<Path>   // read for context only — all findings → Adjacent Observations
}
```

**Populate `modified` and `deleted`**:

```bash
# PR mode
gh pr view <n> --json files --jq '.files[] | [.path, .status] | @tsv'
# Local / Staged
git diff --name-status <base>...HEAD
```

- Status `M`, `A`, `C` → `scope.modified`
- Status `D` → `scope.deleted`
- Status `R` (rename) → new path → `scope.modified`; record old path so the agent avoids critiquing unchanged content as new code

**Populate `testsForModified`**: For each path in `scope.modified`:
1. Mirror into test tree: `Features/Login/LoginViewModel.swift` → `Tests/Features/Login/LoginViewModelTests.swift`
2. Fall back: search for `*Tests.swift` in the same directory
3. Add found paths (that exist on disk) to `scope.testsForModified`

**Populate `related`**: Added during Phase 1 as the agent reads imported files, protocol declarations, and parent views for context.

**Excluded paths** — filter from all sets before finalising:
- `Pods/**`, `Carthage/**`, `.build/**`, `DerivedData/**`, `.swiftpm/**`
- `*.generated.swift`, `*.pb.swift`, `R.generated.swift`
- Any patterns in a `review-excluded-paths` block in `.claude/CLAUDE.md`

#### Scope Banner (mandatory — first output before any findings)

```
Scope: PR #123 · base: main · modified: 7 · tests-for-modified: 3 · deleted: 1 · related: 12
```

For auto-detected PR, prepend a detection notice:
```
Detected open PR #123 (base: main). Run with --local to review uncommitted work instead.
Scope: PR #123 · base: main · modified: 7 · tests-for-modified: 3 · deleted: 1 · related: 12
```

For local mode:
```
Scope: local (base: main) · modified: 4 (3 pushed, 1 uncommitted) · related: 8
```

#### Scope Enforcement

These rules apply throughout Phases 1–3:

- **L1 — file-level**: Any file in `scope.modified` is reviewed in full — every line is in scope, not just changed lines.
- **O1 — quarantine**: Findings against files outside `scope.modified` (and outside coverage checks in `scope.testsForModified`) go into **Adjacent Observations** — a dedicated section at the end of the report labelled *"out of scope for this PR — file separately."*
- **Severity rollup** (`Critical: N | High: N | …`) counts only in-scope findings. Adjacent Observations are excluded.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Viniciuscarvalho/swift-code-reviewer-skill](https://github.com/Viniciuscarvalho/swift-code-reviewer-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
