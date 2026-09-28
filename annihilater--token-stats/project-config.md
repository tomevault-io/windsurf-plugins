---
trigger: always_on
description: This document provides context for AI agents working on the Tokscale project.
---

# AI Agent Guidelines

This document provides context for AI agents working on the Tokscale project.

## Maintaining AGENTS.md Files

When updating AGENTS.md files, follow these principles:

- **No hardcoded counts** — Don't write "10 crates" or "5 modules"; these become outdated instantly
- **No exhaustive lists** — Prefer dynamic commands (`ls crates/`) over maintaining complete lists
- **Document constraints, not descriptions** — Focus on non-obvious behaviors, gotchas, and cross-crate dependencies
- **Use nested AGENTS.md** — Place crate-specific details in `crates/{name}/AGENTS.md`, not here
- **Verify before documenting** — Grep/read the code to confirm claims are accurate
- **Delete outdated info** — Outdated docs are worse than no docs

## Authoring PR/Issue Content via `gh` CLI

When writing PR bodies, issue bodies, or comments through `gh pr create`, `gh issue comment`, etc.: **prefer `--body-file` with a written-out file over inline `--body` heredocs.**

Why: inline `--body "$(cat <<'EOF' ... EOF)"` patterns lead to recurring mistakes where backticks are written as `` \` `` (incorrectly escaping them inside a single-quoted heredoc where no escaping is needed). The literal `` \` `` then renders in GitHub markdown as backslash + backtick instead of inline code formatting.

**DO:**

- Write the body to `/tmp/pr-body.md` (or similar) with the `Write` tool, then `gh pr create --body-file /tmp/pr-body.md`
- For comment edits: `gh api -X PATCH repos/<owner>/<repo>/issues/comments/<id> -F body=@/tmp/comment.md`

**DON'T:**

- Use `gh pr create --body "$(cat <<'EOF' ... \` ... \` ... EOF)"` — single-quoted heredoc means backticks are already literal; escaping with `` \` `` is wrong and renders incorrectly.
- Use double-quoted heredoc for bodies containing backticks unless you intend command substitution.
- Hard-wrap paragraphs at ~80 columns inside PR/issue/comment bodies. GitHub's markdown collapses single newlines into spaces so the rendered output looks fine, but the **raw markdown view is what reviewers and authors edit in**, and mid-sentence line breaks read as if the prose was chopped. Write each paragraph as one continuous line and let the renderer wrap it. Same rule for blockquotes, list items that span multiple lines, and table cells. Hard wraps are still fine inside fenced code blocks where preserving line layout matters.

This applies to all GitHub-content authoring through the CLI — PR bodies, issue bodies, comments, edits. Commit message bodies should also follow this rule: write prose paragraphs as continuous lines, not hard-wrapped at 80 columns.

## Git Identity & Merge Discipline

- Before any commit, inspect the effective Git identity (`git config user.name` / `user.email`) and remotes. If the identity does not match the contributor or expected automation account for the current branch, stop and ask for confirmation.
- Never commit as worker/agent identities such as `worker1`, `worker2`, `worker3`, or `*@example.invalid`.
- When merging pull requests through `gh`, use merge commits (`gh pr merge --merge ...`) unless the user explicitly requests another merge strategy.
- Before merging, verify the merge commit title is the intended conventional PR title and does not contain worker/agent/internal review jargon.

## Commit Message Convention

```
<type>: <description>

[optional body]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `refactor` | Code refactoring (no behavior change) |
| `docs` | Documentation only |
| `test` | Adding or updating tests |
| `chore` | Maintenance tasks |
| `perf` | Performance improvements |

### Examples

```
feat: add session branching with /fork command
fix: handle empty response from provider
refactor: extract streaming logic to separate module
docs: update README with new CLI options
```

### Commit Message & PR Title Rules (CRITICAL)

> These rules apply to **both commit messages AND pull request titles**. PR titles become the squash-merge commit message, so they must follow the same conventions.

**DO:**
- Describe the actual change in plain, technical terms
- Keep commits atomic (one logical change per commit)
- Use the format: `<type>(<scope>): <what changed and why>`

**DON'T:**
- Reference internal review labels (P0, P1, P2, etc.) in commits or PR titles
- Mention "Oracle", "audit", "review findings", "hardening" in commits or PR titles
- Use agent-internal jargon: "wave", "hardening", "compliance", "verification pass"
- Bundle multiple unrelated fixes into one commit
- Use vague messages like "fix issues" or "address feedback"

**Good Examples:**
```
fix(lsp): pass server args to stdio spawn command
fix(lsp): convert 1-indexed input lines to 0-indexed LSP positions
fix(gemini): parse SSE data frames instead of raw JSON lines
fix(orchestrator): route provider tools through approval flow
```

**Bad Examples (NEVER do this):**
```
fix: address P0 issues from Oracle review      ❌
fix(hardening): Oracle Round 4 fixes           ❌
fix: audit findings                            ❌
fix: various improvements                      ❌
fix(tui): harden unreleased changes — P0-P3    ❌  (PR title)
fix: hardening wave 1 compliance fixes         ❌  (PR title)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Annihilater/token-stats](https://github.com/Annihilater/token-stats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
