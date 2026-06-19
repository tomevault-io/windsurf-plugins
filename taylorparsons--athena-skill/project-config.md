---
trigger: always_on
description: Turn a PRD into shipped changes using a strict, repeatable ATHENA loop that captures customer requests in docs/requests.md, decisions in docs/decisions.md, and per-feature specs in docs/specs/FEATURE_ID/spec.md (with Sources/Verifies/Implements traceability), then reads docs/PRD.md and docs/progress.txt to execute one small task at a time with validation and session notes. Use when the user wants iterative PRD-driven delivery with explicit task tracking and an audit trail from raw requests to fi
---


# ATHENA Loop

Follow this loop every session.

Ground rules:
- Treat `docs/requests.md` as an append-only log of raw customer requests (inputs).
- Treat `docs/decisions.md` as an append-only decision trail that explains how inputs became PRD requirements.
- Treat `docs/specs/<feature-id>/spec.md` as the human-readable feature spec with traceability tags (Sources/Verifies).
- Treat `docs/specs/<feature-id>/tasks.md` as the task list with traceability tags (Implements).
- Treat `docs/TRACEABILITY.md` as the entry point explaining how to follow the audit trail.
- Treat `docs/PRD.md` and `docs/progress.txt` as the source of truth for execution.
- Do not make any PRD/code changes until the current session’s customer input has been captured as a new `CR-...` entry (verbatim) in `docs/requests.md`.
- If the project is a Git repo, create local commits as tasks are completed with traceability pointers back to `docs/` (see “6.5) Create a Traceable Local Commit”). Never push to a remote unless the user explicitly requests it.
- If a feature involves UI/UX work, offer the `daisy` skill as an option/framework for the task.

## 0) Capture the Customer Request (Input)

At the start of each session, record the current customer request **verbatim**:
- Ensure `docs/requests.md` exists (create it using the template below if missing).
- Append a new entry with a unique ID and timestamp.
- If the request includes secrets (API keys, passwords, tokens), redact them before writing.

If you do not have an explicit customer request for the current session:
- Ask the user for it before changing `docs/PRD.md`.

## 0.25) Capture the Agent / Skill Context (for context resets)

ATHENA does not automatically track which skills/agents were used. To make sessions recoverable after a context reset, explicitly record the execution context in `docs/progress.txt`.

At the start of each session, set these header fields in `docs/progress.txt`:
- `Agent:` (e.g., `Codex CLI`)
- `Model:` (if known)
- `Skills:` (e.g., `athena`, plus any other skills you actually used during the session)

## Task ↔ Skill Mapping Convention

To preserve a clear mapping between the task you are working on and the skill(s) you used, annotate the task lines in `docs/progress.txt` with a `Skills:` tag.

Format (recommended):

```text
- <task line...> (Skills: athena, <other-skill>, ...)
```

Rules:
- Every task listed under `IN PROGRESS` MUST include `Skills: ...`.
- If you used additional skills mid-task, update that task’s `Skills:` list before ending the session.

## 0.5) Capture Decisions When You Interpret or Change Scope

Whenever you:
- Interpret ambiguous wording,
- Choose between options,
- Decide tradeoffs,
- Change scope,
- Update `docs/PRD.md` in a way that is not a verbatim copy of the request,

Append a decision entry to `docs/decisions.md`:
- Use a unique ID (`D-YYYYMMDD-HHMM`).
- Reference the relevant request IDs (`CR-...`).
- Reference the PRD section(s) impacted (e.g., heading name).
- Keep it short, factual, and testable.

## 1) Ensure Source-of-Truth Files Exist

Check for these files first:
- `docs/requests.md`
- `docs/decisions.md`
- `docs/TRACEABILITY.md`
- `docs/PRD.md`
- `docs/progress.txt`

If any file is missing, create it before doing anything else.

When creating missing files:
- Keep `docs/PRD.md` minimal and factual. Do not invent requirements.
- Initialize `docs/progress.txt` using the template in the “Progress Log Template” section.
- Initialize `docs/requests.md` using the template in the “Customer Request Log Template” section.
- Initialize `docs/decisions.md` using the template in the “Decision Log Template” section.
- Initialize `docs/TRACEABILITY.md` using the template in the “Traceability Entry Point Template” section.
- If the project is already a Git repo with existing history and these audit files were missing, generate a historical audit trail from Git (see “1.25) Bootstrap Historical Audit from Git History”).

## 1.25) Bootstrap Historical Audit from Git History (for adopted repos)

If you are adding ATHENA to an existing project that:
- has Git enabled (a `.git/` directory), and
- is not already using the ATHENA audit files,

create a **historical audit trail derived from Git history** so there is an explicit “before ATHENA” record.

Rules:
- Do **not** fabricate customer requests. Git history is not “verbatim customer input”.
- Keep customer inputs in `docs/requests.md` as verbatim going forward (starting with the current session’s request).
- Store derived history in `docs/audit/git-history.md` and clearly label it as derived.

Preferred implementation:
- Run the bundled script: `python3 <CODEX_HOME>/skills/athena/scripts/bootstrap_git_audit.py --out docs/audit/git-history.md`
- Then update `docs/TRACEABILITY.md` to include a pointer to `docs/audit/git-history.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taylorparsons/athena-skill](https://github.com/taylorparsons/athena-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
