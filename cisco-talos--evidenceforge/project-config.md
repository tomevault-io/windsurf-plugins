---
trigger: always_on
description: This document provides AI coding agents with everything needed to write consistent, idiomatic code for the EvidenceForge project.
---

# AGENTS.md - EvidenceForge

This document provides AI coding agents with everything needed to write consistent, idiomatic code for the EvidenceForge project.

## Project Overview

EvidenceForge generates realistic synthetic security logs for cybersecurity threat hunting training and research. The system uses a two-phase hybrid architecture:

**Phase 1 - Scenario Creation (Skill-assisted):** Claude Code Skills guide users through scenario creation via structured interviews. Skills research TTPs via MITRE ATT&CK, expand high-level descriptions into detailed execution plans, and output structured YAML scenario files with companion research markdown.

**Phase 2 - Log Generation (Deterministic):** Generation engine executes the detailed scenario plan WITHOUT any LLM calls, producing large-scale, temporally consistent datasets across multiple log formats (Windows Event Logs, Zeek, Syslog, Snort/Suricata, web logs) with coordinated cross-references (matching LogonIDs, PIDs, session data).

This architecture combines LLM flexibility/realism with deterministic speed, cost-efficiency, and reproducibility.

**Key Principle:** The `eforge` CLI is a deterministic tool. Creative/interactive work happens through Claude Code Skills, not built-in LLM calls. Phase 2 is a deterministic renderer that executes the plan. Never call LLMs during generation. LLM integration is not built-in; scenario creation uses Claude Code Skills.

**Storyline Events (Phase 8.4):** Storyline entries use typed `events` lists, not free-text keyword matching. Each event has a `type` field (`process`, `logon`, `connection`, `ssh_session`, etc.) with per-type validated fields. The `activity` field is documentation only (for GROUND_TRUTH.md). See `docs/reference/scenario-reference.md` for the full event type reference.

**Baseline Realism:** The baseline engine includes: Hawkes self-exciting temporal model for bursty user activity (parameters derived from persona risk_profile), periodic+jitter timing for system/service traffic, day-of-week variation (Monday login storms, weekend near-zero), 26 legitimate lateral movement patterns (backup, monitoring, AD replication, app→DB, etc.), process→network correlation (browsers→HTTPS, DB clients→SQL, etc.), enriched stale account noise (Kerberos failures, lingering tasks, service startup failures), network-level red herrings (suspicious DNS, unusual outbound, scan overlaps), Linux syslog depth (18 categories including SSH login/key exchange, apt/dnf, systemd timers, logrotate, journald), diversified command pools with per-user parameterization, and entity lifecycle validation (boot time tracking, PID existence checks). Lateral movement patterns are conditional on environment topology — assign `roles` to systems to enable specific patterns.

**Causal Expansion Engine:** The `CausalExpansionEngine` (`src/evidenceforge/generation/causal/`) auto-generates prerequisite and consequent events via composable rules. DNS lookups before TCP connections, Kerberos/DC-bundle TGT/TGS evidence before domain logons, ProcessAccess after lsass injection, and supplementary audit events from command-line patterns are all handled automatically — scenario authors should NOT manually specify these as prerequisites. Authors CAN still specify these event types when they are part of the attack narrative itself (e.g., DNS tunneling, golden ticket forging). The validator warns on potentially redundant manual specifications. See `docs/ARCHITECTURE.md` § Causal Expansion Engine for implementation details.

## MANDATORY: Project Memory Workflow

**CRITICAL: Read this section first before doing ANY work on this project.**

This project uses `TODO.md` as the durable roadmap and backlog. It also uses
tracked worklogs under `docs/worklog/` for multi-session effort memory. This is
designed to preserve agent continuity without forcing every branch to edit the
same high-conflict roadmap file.

### Required Workflow for Every Session

1. **START OF SESSION (BEFORE ANY WORK):**
   - **ALWAYS read `TODO.md` first** to understand:
     - What phase/milestone the project is in
     - What durable backlog items remain
     - Which worklogs contain active handoff context
   - If `TODO.md` references a relevant worklog, read that worklog before
     changing code.
   - If `TODO.md` doesn't exist, create it with the initial durable roadmap
     based on the PRD.

2. **DURING NORMAL TASK WORK:**
   - Do **not** edit `TODO.md` just to mark a task as started, in progress, or
     completed.
   - For one-shot fixes, use commits, PR descriptions, and final response notes
     as the task record.
   - For multi-session efforts, assessment loops, investigations, or branch-local
     progress that future agents need, create or update one focused worklog under
     `docs/worklog/`.
   - Use one worklog per effort, named `YYYY-MM-DD-<effort-slug>.md`.

3. **WHEN TO UPDATE `TODO.md`:**
   - Add, remove, or reprioritize durable backlog items.
   - Record a milestone-level completion or major roadmap pivot.
   - Reconcile roadmap state during release/integration work.
   - Replace detailed progress history with a short pointer to a worklog or
     changelog entry.

4. **WHEN ADDING NEW WORK:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cisco-Talos/EvidenceForge](https://github.com/Cisco-Talos/EvidenceForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
