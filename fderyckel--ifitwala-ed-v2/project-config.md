---
trigger: always_on
description: This repository is a production-grade, multi-tenant Education ERP built on **Frappe Framework v16** with a **Vue 3 + Tailwind + frappe-ui SPA**.
---

File: AGENTS.md

# AGENTS.md — Ifitwala_Ed Repository Constitution

This repository is a production-grade, multi-tenant Education ERP built on **Frappe Framework v16** with a **Vue 3 + Tailwind + frappe-ui SPA**.

This file is the **root constitution** for all agents working in this repository.
Nested `AGENTS.md` files may add local rules, but they must never weaken this contract.

---

## 0. Prime Directive (Non-Negotiable)

> **Do not invent. Do not assume. Do not drift.**

- Never invent field names, DocTypes, schemas, routes, payloads, permissions, defaults, or workflow rules.
- Never rely on memory of older repo versions.
- Always work from files currently present in the workspace.
- If required files, contracts, or schemas are missing: **STOP. ASK. WAIT.**

---

## 0.1 Execution Priority Order (Non-Negotiable)

When evaluating any change, think in this order:

1. Product UX
2. Security and permissions
3. Data integrity and workflow invariants
4. Multi-tenant isolation
5. High concurrency and scalability
6. Framework correctness
7. Code style and cleanup

If a change improves code elegance but weakens product UX, security, data integrity, or concurrency safety, reject it.

---

## 0.2 Product Mandate (Non-Negotiable)

Agents must operate with a product manager mindset and reduce friction for real users:

- teachers
- admissions staff
- academic staff
- guardians
- students
- administrators

Rules:

- Prefer in-product workflows over CLI/manual operator steps.
- Eliminate avoidable navigation and context switching.
- Surface actions where users already work.
- Silent UI dead ends are defects.
- Blocked actions must explain:
  - why the action is blocked
  - what the user should do next

Before implementing workflow/UI changes, ask:

- Is this the lowest-friction path?
- Is the next action obvious?
- Does the UI preserve user context?
- Is failure actionable instead of silent?

---

## 0.3 Framework Baseline (Non-Negotiable)

- Runtime baseline is **Frappe Framework v16**.
- Any workflow, setup, patch, or instruction that pins framework version must target **`version-16`**.
- For Frappe list/query pagination in Python, JS, and typed contracts, use `limit`, never `limit_page_length`.

---

## 0.4 Local Environment Communication Rule

- Codex is running on the user's local machine in this repository, not on a remote production server.
- This local Codex environment is the user's MacBook Air laptop and does not provide Frappe `bench`; do not attempt `bench` commands here.
- Do not add boilerplate disclaimers that the repo `.venv` does not contain `frappe`, that `bench` is not on `PATH`, or that the current shell is not the server.
- If verification is blocked, report only the concrete blocker for the command you attempted and keep the note short.
- Do not treat local shell path differences as architecture insight or as a reason to add generic environment caveats to the closeout.

---

## 1. Operating Discipline

### 1.1 Mandatory Workflow

For non-trivial tasks, agents MUST:

1. Restate the task precisely.
2. Identify exact files to inspect and likely files to change.
3. Summarize relevant doc/code/contract constraints.
4. List key risks; for failing tests, identify the first non-test code path in the traceback and fix shared factory, fixture helper, reference-data, or harness-bootstrap failures before editing production workflow code:
   - UX regressions
   - permission leaks
   - data integrity/workflow regressions
   - concurrency/performance regressions
5. Stop for explicit approval before:
   - structural refactors
   - schema changes
   - cross-module behavior changes
6. Execute only approved work.

No opportunistic cleanup. No scope creep.

### 1.2 Generated Artifact And I18n Discipline

- Do not commit or leave behind oversized generated audit artifacts, scan dumps, or machine-produced markdown unless the user explicitly asks for them to live in the repo.
- Before writing generated docs under `ifitwala_ed/docs/`, prefer concise human review outputs over exhaustive raw dumps.
- If a generated artifact is likely to be large, split it, summarize it, or keep it outside the tracked repo workflow.
- Treat any single generated markdown file approaching repository or tool limits as a process failure to avoid, not a lint issue to discover later.
- For i18n, translation functions must receive stable literal source strings only.
- `_()` and `__()` are reserved for translation only.
- Never assign to `_` or `__`, and never use `_` as a throwaway variable, tuple-unpack target, or temporary alias.
- Never pass variables directly to `_()` or `__()`.
- Never use f-strings, template literals, or string concatenation as the translatable source sentence.
- When dynamic data is required, use a literal source string with named placeholders, then format after translation.

---

## 2. Architectural Authority & Drift Control

- Architecture is explicitly locked.
- Markdown under `ifitwala_ed/docs/` is authoritative.
- Code and documentation must never silently diverge.
- If requested behavior changes:
  - identify the canonical doc
  - update docs together with code when approved

Drift is a bug.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fderyckel/ifitwala_ed_v2](https://github.com/fderyckel/ifitwala_ed_v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
