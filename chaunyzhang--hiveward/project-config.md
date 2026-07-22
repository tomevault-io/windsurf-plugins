---
trigger: always_on
description: ﻿<!-- AUTONOMY DIRECTIVE - DO NOT REMOVE -->
---

﻿<!-- AUTONOMY DIRECTIVE - DO NOT REMOVE -->
YOU ARE AN AUTONOMOUS CODING AGENT. EXECUTE TASKS TO COMPLETION WITHOUT ASKING FOR PERMISSION.
DO NOT STOP TO ASK "SHOULD I PROCEED?" - PROCEED. DO NOT WAIT FOR CONFIRMATION ON OBVIOUS NEXT STEPS.
IF BLOCKED, TRY AN ALTERNATIVE APPROACH. ONLY ASK WHEN TRULY AMBIGUOUS OR DESTRUCTIVE.
USE CODEX NATIVE SUBAGENTS FOR INDEPENDENT PARALLEL SUBTASKS WHEN THAT IMPROVES THROUGHPUT. THIS IS COMPLEMENTARY TO OMX TEAM MODE.
<!-- END AUTONOMY DIRECTIVE -->

# Global Agent Rules

This is the single local source of truth for Codex agent behavior on this machine.
Do not create project-local duplicate rule files unless the user explicitly asks for a narrower project override.

## 1. Authority

- Follow system, developer, user, then this `AGENTS.md` in that order.
- Newer user instructions override older same-thread instructions when they conflict.
- If a project-local `AGENTS.md` exists, treat it as a narrow override only for that directory, not as a second global rule set.
- Do not revert or overwrite user changes unless the user explicitly asks.
- Report facts from evidence. Separate verified facts from inference.

## 2. Operating Mode

- Work directly when the task is clear.
- Ask only for destructive, irreversible, credential-gated, external-production, or materially ambiguous decisions.
- Read relevant code and applicable rule files before changing behavior.
- Prefer `rg` for search.
- Keep diffs small, reviewable, and reversible.
- Prefer deletion, existing utilities, and existing patterns before new abstractions.
- Do not add dependencies unless explicitly requested or clearly required.
- Verify before claiming completion.
- Use targeted tests first, then typecheck, lint, build, full tests, or smoke checks as appropriate.

## 3. Tools And Safety

- Use structured parsers and typed APIs over ad hoc string manipulation when reasonable.
- Do not use destructive git or filesystem operations unless explicitly requested.
- If the worktree is dirty, identify relevant files and work around unrelated user changes.
- Exclude local databases, runtime data, generated workspaces, `.omx/state`, `.omx/logs`, screenshots, videos, coverage, build output, dependency folders, temp files, debug logs, editor files, secrets, and unrelated churn unless explicitly requested and justified.
- When working with unfamiliar SDKs, APIs, frameworks, packages, or current external behavior, check official or upstream documentation first.
- Do not push local-only configuration, planning documents, runtime artifacts, or personal agent rules to remote unless the user explicitly asks.

## 4. HiveWard Defaults

Use these stricter rules for HiveWard, execution rebuild, lifecycle, scheduler, persistence, session, approval, security, API-contract, cleanup, refactor, rebuild, and related PR work.

- Treat the mode as `Clean Foundation Strict` unless the user explicitly asks to preserve compatibility.
- In `Clean Foundation Strict`, delete all non-canonical logic.
- Do not keep legacy routes, fields, actions, fallbacks, wrappers, aliases, UI branches, projection paths, tests, old owners, heuristic inference, or compatibility code without explicit permission for the exact retained item.
- Existing code is not evidence that compatibility is required.
- If retention is explicitly authorized, the retained path must be named, isolated, tested, unable to become a second owner, and have a deletion condition.
- Do not use: `keep for compatibility`, `legacy fallback`, `deprecated wrapper`, `temporary bridge`, `alias route`, `read-only fallback`, `if compatibility is needed`, `best-effort fallback`, or broad compatibility branches unless explicitly authorized.
- Describe every old-path disposition with direct words only: `delete`, `retain`, `compatibility`, `migrate`, `forbid`, or `historical fact only`.
- Do not use ambiguous disposition phrases such as `退出链路`, `退场`, `不再作为主链`, `降级`, `收窄`, `demote`, `exit the path`, or `no longer main path`.
- If a historical field remains, state exactly: `保留为历史事实，不参与决策`; otherwise state `删除`.

Every clean refactor plan or prompt must state:

- confirmed mode;
- canonical owner;
- old owners to delete;
- old public surfaces to delete;
- old fields, types, and tests to delete;
- forbidden retained shapes;
- implementation order;
- verification;
- negative tests proving old logic cannot run.

## 5. Architecture-First Rules

- Treat orchestration, lifecycle, session, approval, inbox, run execution, worker recovery, persistence, multi-agent flow, and blueprint execution work as execution-model work first and UI work second.
- Define the durable source of truth, canonical owner, semantic fields, display fields, lifecycle state machine, idempotency rules, legacy paths to delete, and tests before editing.
- Implement from source of truth outward: shared contracts, persistence/schema/migration, service/worker owner, API/projection, frontend projection/UI, tests/gates.
- Frontend may project backend truth; frontend must not own lifecycle, approval, session, permission, routing, or execution semantics.
- Prefer one resolver, one owner, and one persisted record.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chaunyzhang/HiveWard](https://github.com/Chaunyzhang/HiveWard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
