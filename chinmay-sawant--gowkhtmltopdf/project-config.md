---
trigger: always_on
description: > This file is the conventions ledger for every coding agent working in this
---

# AGENTS.md - gowkhtmltopdf

> This file is the conventions ledger for every coding agent working in this
> repo (opencode, grok, gemini, codex, antigravity/agy, claude). Read it at
> session start. It encodes lessons from a 490-session audit of our sibling
> projects plus everything learned shipping gowkhtmltopdf itself through
> v0.2.x: the mistakes, repeat work, and avoidable waste we already paid for,
> so we do not repeat them here.

## Project

Pure-Go HTML-to-PDF engine and wkhtmltopdf-style work-alike, plus an
HTML-to-image rasterizer. Two static binaries (`cmd/gowkhtmltopdf`,
`cmd/gowkhtmltoimage`) and a Go library (`Document` / `ImageDocument` at
the repo root). The product is print-oriented structured documents
(invoices, receipts, certificates), not Chrome visual parity. No JavaScript.
No CGO.

Module: `github.com/chinmay-sawant/gowkhtmltopdf`.
GitHub repo: `https://github.com/chinmay-sawant/gowkhtmltopdf`.
Default branch: `master`. Current version: `VERSION` (single line, injected
into both binaries via ldflags).

Pipeline order, owned by `internal/`: load -> html parse -> css cascade ->
layout -> paginate -> paint -> pdf write. `internal/convert/` orchestrates;
`internal/pdf/` is the version-aware writer; `internal/imageout/` shares the
pipeline for PNG/JPEG.

## Todo protocol - response-only (mandatory)

Every agent must manage todos live in the API response, not on disk. This is
the first action on any task, before any reads, edits, or other tool calls.

1. **Create todos in the response via API before any work.** On receiving any
   task (feature, fix, docs, question with multi-step work), immediately call
   the todo API (`todowrite` or equivalent) to publish the plan as todos. Do
   not start work until the todo list is visible in the API response.
2. **Show current todos in every response.** Each assistant turn must render
   the current todo list with status markers (`pending`, `in_progress`,
   `completed`, `cancelled`) and clearly highlight which item is
   `in_progress`. The todo list is the live progress bar for the user.
3. **Do not store todos on disk.** Do not create `TODO.md`, `todos.json`, or
   any other todo-tracking file. Todos live only in the API response state.
4. **Keep response todos updated as you go.** Mark items `in_progress` when
   started and `completed` when finished. If scope changes, update the list
   immediately.
5. **Completion requires todos to show done.** A task is done only when all
   todos show `completed` and you have sent a final summary stating what
   shipped.

If the todo API is unavailable, state that in the response and list todos
inline as a fallback - still do not write a file.

## Golden rules

1. **No git commands without explicit permission.** Never run `git add`,
   `git commit`, `git push`, `git restore`, `git clean`, `git reset`, or
   `git stash` unless the user asks. Subagent prompts carry this ban by
   default.
2. **No em dashes ("—") in any written output, docs, or commit messages.**
   Use plain hyphens or restructure. This includes this file.
3. **Commit at session end; never leave a dirty tree.** Uncommitted work is
   the top cause of cross-session rework. If interrupted, record what landed.
   Ask before committing if the user has not authorized it.
4. **Branch naming:** lowercase `feature/`, `fix/`, `chore/`, or
   `docs/<short-description>`. Verify the branch name before the first commit
   (a typo'd `chore/frontend-udpates` once lived across 3 commits and a PR).
5. **PRs** use `skills/PR/PR_TEMPLATE.md`; issues use
   `skills/PR/ISSUE_TEMPLATE.md`. Whenever a user asks to raise, update, or
   review a PR, read the PR template first. Generate the diff table with
   `bash scripts/pr-diff-stat.sh <base-ref>` and paste the complete output
   into the body. Body files live at `plans/PR/pr-<short-slug>.md` and must
   stay in sync with the GitHub PR. PRs require self-assignment and at least
   one label.
6. **Checklists are live ledgers:** phase files under `plans/<version>/`
    close rows `[x]` in the same change that implements them, and only when
    the gate actually passed. Never mark `[x]` from intent. When you create
    or finish a plan under `plans/`, update `knowledge-base/` in the same
    session so the local narrative matches what shipped.
7. **Answer from knowledge-base, prove with code.** Start in
   `knowledge-base/wiki/index.md`, then open the real source under
   `internal/`. A KB hit is a starting point, not proof. If KB and code
   disagree, trust the code, fix the KB, answer from the code. For prose
   claims (performance numbers, fidelity statements), the committed reference
   is `documentation/*.md`, and `make claim-scan` polices forbidden claims
   there.
8. **Unslop all writing.** Before writing plans, docs, knowledge-base pages,
   PR/issue text, commit messages, or user-facing replies, apply `/unslop`.
   Cut puffery, em dashes, chatbot filler, synonym cycling, title-case
   headings.
9. **Feynman every explanation and document.** Whenever you respond with an
   explanation or produce documentation (plans, `documentation/`,
   knowledge-base pages, README sections, PR bodies, design notes), apply
   `skills/feynman/SKILLS.md`: plain words a smart 12-year-old could retell,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chinmay-sawant/gowkhtmltopdf](https://github.com/chinmay-sawant/gowkhtmltopdf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
