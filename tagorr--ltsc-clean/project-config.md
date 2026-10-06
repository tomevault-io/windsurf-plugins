---
trigger: always_on
description: **Scope:** Windows 11 Enterprise LTSC 2024 (`EditionID=EnterpriseS` or `IoTEnterpriseS`, `24H2`, build `26100+`, strict display-version gate). We manage only the install scripts.
---

# AGENTS.md

**Scope:** Windows 11 Enterprise LTSC 2024 (`EditionID=EnterpriseS` or `IoTEnterpriseS`, `24H2`, build `26100+`, strict display-version gate). We manage only the install scripts.

## Allowed to edit

`SetupComplete.cmd`, `BaselinePolicies.txt`, `PreOOBE.cmd`, `BootstrapLocalAdmin.ps1`, `ConfigureDefenderPrivacy.ps1`, `CreatePrimaryAdmin.ps1`, `ValidateSecrets.ps1`, `README.md`, `DECISIONS.md`, `SECURITY.md`, `CONTRIBUTING.md`, `docs/AUDIT_CHECKLIST.md`, `docs/INTERACTION_CONTRACT.md`, `docs/VALIDATION.md`, `docs/GUIDE.md`, `docs/QUICK_START.md`, `docs/PIPELINE_FLOW.md`, `docs/OPERATIONS.md`, `docs/TROUBLESHOOTING.md`, `docs/ARCHITECTURE.md`.

**Owner-controlled areas** (default: out of scope for agents)

The allow-list in this document is the default scope for agent-initiated edits. The Owner may explicitly authorize exact additional paths for one specific task/PR; that authorization overrides the default restriction only for that task and must be documented in the PR description. A one-off authorization does not add the path to the permanent default allow-list. Permanently adding a path to the default scope requires a separate update to this document.
Files under `tools/` and `.agents/skills/` are owner-controlled. Agents must not create, modify, or delete files in these areas unless the current task explicitly includes those paths in its allow-list.


**Forbidden:**

- Any edits by agents to files that are neither listed in the “Allowed to edit” section above nor explicitly authorized by the Owner for the current task/PR under the exception above.
- Adding a path to the permanent default agent edit scope without, in the same PR:
  - adding it to the “Allowed to edit” list in `AGENTS.md`, and
  - (recommended) backing the change with an ADR that explains why the path belongs in the permanent default scope.
- Carve-out (Interaction Contract): when temporary Codex-created files are genuinely needed, they MAY be created only under the repository root's `.codex_tmp\` directory. This carve-out does not allow creating new files elsewhere.

## Roles and responsibilities

* **Owner (maintainer/operator):** defines the scope for each change, runs Codex CLI locally, reviews diffs, applies changes, performs all state-changing Git actions (commit, push, PR, merge), and remains accountable for final behavior and security posture.
* **ChatGPT:** helps draft prompts, audits, and documentation text. Has no direct access to the repo working copy, cannot run commands or Git actions, and must not claim that tests were executed.
* **Codex CLI:** edits files in the local working copy within the allowed scope, produces minimal diffs, follows `docs/INTERACTION_CONTRACT.md`, does not perform state-changing Git actions, and does not expand scope without explicit instruction. Read-only Git commands (for example `git status`, `git diff`) are allowed when needed for situational awareness.
* **CI (GitHub Actions):** runs automated checks on PRs (for example EOL/BOM/NUL guardrails and ASCII-only scripts via "ASCII Only Guard") and reports status only. CI does not replace human review and does not modify repository content.

## Invariants

* **No immediate reboots** inside `SetupComplete.cmd`. Reboot requirements are signaled only via `%WINDIR%\Panther\_needs_reboot.flag` (`Panther flag`) when `RC ∈ {3010, 1641}` or `ALWAYS_REBOOT_AFTER_FIRST_LOGON=1`. `SetupComplete.cmd` never calls `shutdown.exe`.
* Documentation must match actual behavior (paths, logs, steps). Use `README.md` as the front door, `docs/GUIDE.md` as the documentation router, `DECISIONS.md` for rationale and trade-offs, `SECURITY.md` for security posture, and the relevant `docs/*` owner document for procedural or operational detail.

### Tracked Windows runtime code

These rules govern the installation scripts that run on the supported Windows target. They do not prescribe how Codex executes tools.

* Tracked PowerShell scripts and PowerShell commands launched by tracked runtime code must remain compatible with **Windows PowerShell 5.1**.
* When tracked PowerShell runtime code uses `reg.exe`, use classic registry paths such as `HKLM\...` and inspect `$LASTEXITCODE`. PowerShell registry cmdlets use provider paths such as `HKLM:\...`. Preserve the existing `Reg-Del` idempotency semantics in `CreatePrimaryAdmin.ps1`, including `{0,2}` as successful delete outcomes.
* In tracked `.cmd` files, `EnableDelayedExpansion` is forbidden. Use plain `%VAR%` expansion and implement branching via labels and subroutines (`goto`, `call :sub`) without relying on delayed expansion.
* **CMD parse-time expansion:** inside any parenthesized `(...)` block, it is forbidden to read `%VAR%` for variables that may be set/modified within the same block (including changes caused by `call :sub` invoked from that block). Only acceptable fixes: move the read/log/branch outside the `(...)` block, or use CALL-expansion with `%%VAR%%` (example: `call :log "resolved_path=%%OUT_PATH%%"`).

## Codex repository contract

`docs/INTERACTION_CONTRACT.md` is the canonical contract for repository-operation safety and integrity.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tagorr/ltsc-clean](https://github.com/tagorr/ltsc-clean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
