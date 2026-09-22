---
trigger: always_on
description: 1. Read `docs/PRODUCT_BASELINE.md` before making product changes. It is the only product baseline. Read-only tasks and instruction/documentation maintenance do not require product-baseline reading unless a product decision depends on it. Reuse already-read, unchanged context.
---

# OrdinConn Agent Rules

1. Read `docs/PRODUCT_BASELINE.md` before making product changes. It is the only product baseline. Read-only tasks and instruction/documentation maintenance do not require product-baseline reading unless a product decision depends on it. Reuse already-read, unchanged context.
2. Never restore features or product definitions from the previous OrdinConn repository.
3. Make small, testable changes and inspect the active implementation before editing it.
4. Keep Evidence distinct from model inference and keep every Published Signal traceable to Evidence.
5. User-funds operations always require explicit approval. V0.1 contains no real-money execution.
6. Never collect private keys, seed phrases, or password-field contents.
7. Route every model through Model Gateway and every data source through Connector Registry.
8. A Trade Proposal must pass through Approval before any execution adapter.
9. Keep core crates transport-agnostic and keep React behind typed IPC contracts.
10. Use the approved user-provided purple, black, and yellow brand system and locale keys for all formal UI copy.
11. For product code changes, run Rust tests, TypeScript tests, typecheck, and Rust/frontend builds before declaring completion. Run desktop packaging when packaging, native integration, or release delivery is affected. For read-only or instruction/documentation-only tasks, validate the relevant instructions, links, and diff instead. Reuse successful checks for identical code, dependencies, commands, and relevant environment; rerun affected checks after changes and report failures or unverified criteria explicitly.
12. Push only when the user explicitly asks or when the issue-first public development workflow below already authorizes the same scoped task. Never force-push.

## Task scope and Skills

- Use a Skill when explicitly requested or when its capability directly matches the task and actual technology. A keyword, file extension, dev-server start, or progress question alone is insufficient. Reading a Skill for audit does not activate its workflow.
- Load only task-relevant references. Reuse unchanged instructions already in context; reread when content changes or needed context is missing. Freetower, Figma, Vercel, and analytics artifact workflows apply only when that project, tool, or deliverable is actually in scope. OrdinConn desktop validation follows the real Tauri application and typed IPC path.
- For clear, authorized, reversible work, inspect the implementation, make the smallest useful change, and continue without design, per-section, execution-method, or continuation approval prompts. Preserve an explicit plan-only, review-only, or approval-first boundary. Ask about material missing information or scope changes that cannot be resolved from the request and existing context.
- Existing authorization applies to the same scope and action; it does not authorize funds operations, destructive data changes, sensitive access, new external communications, or publication. Obtain any missing explicit approval for those actions before execution. Never infer consent from silence. The funds, privacy, Approval, and no-push rules above remain binding.
- Choose one workflow to coordinate planning, debugging, and verification. Share its evidence with supporting Skills; do not repeat intake, full test suites, or review of an unchanged diff. Diagnose routine dependency/test failures within scope; escalate when progress requires missing authority or a material user decision.
- Use existing components, dependencies, Model Gateway, and Connector Registry. Create worktrees only when isolation is needed or requested. Verify file ownership and destination before writes; preserve unrelated changes. Generate persistent plans only when useful to the requested deliverable. Commit only when requested or established project policy authorizes it; do not present merge/push menus for ordinary task completion.

## PUBLIC DEVELOPMENT LOG POLICY

- After an engineering-significant task, append a sanitized `Public Interaction Summary` to `docs/devlog/YYYY-MM-DD.md` with: Timestamp, Stage, User Goal, inspected areas, changes, files changed, problems, solution, commands/tests, result, remaining risks, next step, and a Codex engineering note.
- Never copy a full prompt, private conversation, credential, token, cookie, session, password, account identity, customer data, private contact detail, exact personal address, restricted asset, or raw machine home path into the public log.
- Use current code, current documentation, Git history, and fresh verification as truth sources. Keep `Implemented`, `Designed`, `Verified`, and `Blocked` distinct; fixture coverage never substitutes for real-environment evidence.
- Scheduled public-log sync may stage only `docs/`, `social/x/drafts/`, `.github/ISSUE_TEMPLATE/`, and `.github/PULL_REQUEST_TEMPLATE.md`. It must never stage Actions workflows. Product code and sync/security scripts must use the verified task-completion commit path, never the two-hour documentation job.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Btkkgo/OrdinConn](https://github.com/Btkkgo/OrdinConn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
