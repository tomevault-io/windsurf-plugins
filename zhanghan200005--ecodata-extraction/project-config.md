---
trigger: always_on
description: This file applies to the entire repository. A more specific `AGENTS.md` or
---

# EcoEvidence repository instructions

## Scope and instruction priority

This file applies to the entire repository. A more specific `AGENTS.md` or
`AGENTS.override.md` in a subdirectory may refine these rules for that subtree.

## Project mission

Build **EcoEvidence / EcoData Extraction**, a Literature-to-Data system that
turns scientific PDFs into structured, reviewable records while preserving a
traceable link from every extracted field to its source document, page, and
evidence block.

The target workflow is:

```text
literature discovery -> PDF acquisition -> text/OCR/table parsing
-> embedding and vector retrieval -> schema-guided RAG extraction
-> rule-based quality checks -> human review -> traceable export
```

The public repository must distinguish verified implementation from plans.
Never describe a capability as complete unless code, a runnable path, and
proportionate tests support the claim. Label incomplete work `Planned`,
`Prototype`, or `Experimental` in public documentation.

## Authoritative checkout

- GitHub repository: `ZhangHan200005/EcoData_Extraction`
- Owner's normal local checkout: `/Users/zhanghan/Documents/dataFetching`
- Treat the Git root as the project root. Do not create nested clones or extra
  manually managed copies inside the repository.
- Default to the normal local checkout. Do not create or remove worktrees unless
  the user explicitly requests parallel work.
- Never let two active tasks modify the same checkout at the same time. If
  parallel implementation is explicitly requested, use separate managed
  worktrees and separate branches.

## Required start-of-task routine

Before editing:

1. Run `git status -sb` and identify the current branch and upstream.
2. Read `README.md`, `docs/ROADMAP.md`, `docs/ARCHITECTURE.md`, and the files and
   tests relevant to the requested milestone.
3. Inspect existing behavior before proposing replacement code.
4. Report the verified current state, the active milestone, the intended scope,
   and measurable acceptance criteria.
5. If the worktree contains unrelated changes, preserve them and do not stage
   them. Ask only when safe separation is impossible.

## Milestone discipline

- Work on one independently verifiable milestone at a time.
- Use `docs/ROADMAP.md` as the source of truth for milestone status and scope.
- Keep changes small enough to review in one pull request where practical.
- Prefer vertical slices that leave a runnable user path over disconnected
  scaffolding.
- Do not silently expand a milestone into product redesign, paid services, or
  unrelated cleanup.
- At milestone completion, update the Roadmap, architecture documentation,
  public feature-status wording, evaluation results, and known limitations.

## Branch, commit, and PR workflow

- `main` must remain runnable, explainable, and suitable for portfolio review.
- Do not implement directly on `main`.
- Create one branch per milestone from the latest `main`.
- Do not use `codex` in branch names. Prefer:
  - `feature/<milestone>` for product work
  - `fix/<short-description>` for defects
  - `docs/<short-description>` for documentation-only work
- Use concise semantic commit messages such as
  `feat: add versioned embedding retrieval` or
  `docs: define project roadmap and agent rules`.
- Review the final diff before staging. Stage only files belonging to the
  milestone.
- Push the branch and open a Draft PR. All changes enter `main` through a PR and
  passing CI.
- Do not force-push, rewrite shared history, delete branches or data, bypass CI,
  or directly overwrite `main` without explicit user confirmation.

## Repository map

```text
app/              frontend evidence-review workbench
backend/          FastAPI, parsing, retrieval, screening, evaluation, storage
backend_tests/    backend unit and integration tests
demo/             public synthetic PDF and expected evidence
docs/             architecture, Roadmap, and learning documentation
tests/            frontend build and rendered-page tests
data/             local runtime data; database files must remain untracked
```

Keep HTTP routing in `backend/api.py`, workflow orchestration in
`backend/services.py`, data access in `backend/database.py`, and contracts in
`backend/schemas.py`. Avoid putting business logic into the frontend or API
route layer.

## Verification commands

From the repository root, run all of the following before publishing a
milestone unless the change is demonstrably documentation-only:

```bash
npm run lint
npm run test:backend
npm test
git diff --check
```

`npm test` includes a production build and rendered-page tests. When changing
retrieval, extraction, validation, or provenance behavior, add focused backend
tests and at least one end-to-end fixture assertion. Do not weaken or skip tests
to make a change pass.

For documentation-only changes, at minimum run `git diff --check`, validate all
referenced commands and paths against the repository, and confirm the working
tree contains only intended files.

## Data, secrets, and dependency safety

- Never commit API keys, tokens, `.env` files, SQLite databases, caches, logs,
  model artifacts, build output, or local absolute paths containing private
  data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhangHan200005/EcoData_Extraction](https://github.com/ZhangHan200005/EcoData_Extraction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
