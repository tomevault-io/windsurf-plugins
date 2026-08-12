---
trigger: always_on
description: scrum-start.sh           # Entry point — validates prereqs, launches tmux (supports --autonomous)
---

# Maul Team Development Guidelines

## Project Structure

```text
scrum-start.sh           # Entry point — validates prereqs, launches tmux (supports --autonomous)
agents/                  # Agent + 11 sub-agent definitions (top-level: scrum-master, developer, product-owner, requirements-analyst; sub-agents listed in docs/contracts/sub-agents.md)
  scrum-master.md        # Team lead (Delegate mode)
  developer.md           # Developer teammate (PBI pipeline conductor)
  product-owner.md       # PO teammate (autonomous mode; po_mode=agent)
  requirements-analyst.md # Requirement Definition ceremony (interview + mandatory benchmark web search + requirements.md/CLAUDE.md authoring)
  # Per-PBI Integrity stage (5-aspect, Developer-spawned at Round tail): requirement-conformance-reviewer, functional-quality-reviewer, security-reviewer, maintainability-reviewer, docs-consistency-reviewer
  # Sprint-end cross-review is audit-only: 4 whole-repo codebase-audit axes (general-purpose Agent spawns, not named agents)
  # PBI pipeline (per Round): pbi-{designer,implementer,ut-author}, codex-{design,impl,ut}-reviewer
skills/                  # 19 Skills (Scrum ceremonies + pipeline/merge/orchestration tooling + 1 PO acceptance + 1 brief authoring) — YAML frontmatter + Markdown, deployed to target projects via setup-user.sh
  backlog-refinement/    # Refine PBIs from coarse to sprint-ready
  create-brief/          # Co-author docs/product/brief.md with the human (interactive); pre-flight for autonomous launch when no brief exists
  change-process/        # Manage changes to frozen design docs
  codebase-audit/        # Whole-repo 4-axis audit (spec-conformance/logic-defect/redundancy/product-security); IS the Sprint-end cross-review (non-blocking, next-Sprint PBIs) + thin re-check at Integration-Sprint entry
  cross-review/          # Sprint-end audit-only ceremony (runs codebase-audit; non-blocking)
  pbi-pipeline/          # PBI conductor pipeline (orchestrator + references/)
  pbi-escalation-handler/ # SM-side escalation handler
  pbi-merge/             # SM-side per-PBI merge orchestration
  install-subagents/     # Install specialist sub-agents for PBI work
  integration-tests/     # Design-driven systematic integration testing (Integration Sprint)
  uat-release/           # UAT walkthrough, defect routing, and the release decision
  po-acceptance/         # PO-owned demo/UAT verification (autonomous mode)
  requirement-definition/   # Elicit requirements from user
  retrospective/         # Sprint retrospective ceremony
  scaffold-design-spec/  # Create design doc stubs from catalog
  smoke-test/            # Automated test execution
  spawn-teammates/       # Spawn developer teammates for sprint
  sprint-planning/       # Sprint planning and PBI assignment
  sprint-review/         # Sprint review ceremony
.claude/skills/          # Dev-only skills for THIS repo (not deployed to target projects)
  cleanup-audit/         # 8-axis multi-agent repo hygiene audit (read-only)
hooks/                   # Claude Code hooks (status/path/scrum-state/branch-ops guards, stop-dispatch single-entry → dashboard + attention + completion-gate, quality + stop-failure gates, session context, autonomy lib)
  stop-dispatch.sh       # Single Stop entry: forwards payload to dashboard-event + notification-attention (best-effort) then completion-gate
  notification-attention.sh # Notification/Stop/UserPromptSubmit → .scrum/attention.json ("waiting for the human" flag polled by an external UI)
  completion-gate.sh     # Stop gate; mode-dependent block policy (see docs/contracts/agent-interfaces.md § Stop Hook)
  lib/                   # Shared hook helpers (validate, dashboard, autonomy, stop-gate-state)
rules/                   # Cross-cutting context auto-loaded by every Scrum agent (deployed by setup-user.sh to .claude/rules/)
  scrum-context.md       # Team map, SSOT locations, communication protocol, PO seat resolution, uncertainty handling
dashboard/               # Textual TUI dashboard (Python)
  app.py                 # Main TUI application
scripts/                 # Setup and utility scripts
  lib/                   # Shared script helpers (prereq checks)
  setup-user.sh          # Copies agents/skills/hooks/rules to target project
  setup-dev.sh           # Installs dev dependencies (bats, shellcheck, etc.)
  statusline.sh          # Claude Code status line script
  stall-watchdog.sh      # External teammate-stall monitor (non-autonomous mode); launched by scrum-start.sh, nudges SM via tmux on global idle (`stall_watchdog.idle_threshold_minutes`) or when a single in-flight PBI's artifacts+worktree go quiet (`pbi_idle_threshold_minutes`) while the rest of the team stays active
  scrum/                 # SSOT state wrappers (deployed to .scrum/scripts/ by setup-user.sh)
  autonomous/            # Autonomous-PO watchdog (Ralph Loop): watchdog.sh + lib/report.sh
tests/                   # Test suites
  unit/                  # Bats unit tests
  lint/                  # Bats lint tests
  integration/           # Script composition tests
  fixtures/              # Test data (JSON fixtures for validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sohei56/maul-team](https://github.com/sohei56/maul-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
