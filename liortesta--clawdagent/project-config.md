---
trigger: always_on
description: > This file is loaded automatically at the start of every Claude Code session.
---

# Project Configuration — Claude Code MEGA Setup

> This file is loaded automatically at the start of every Claude Code session.
> It serves as the "constitution" — all agents and commands follow these rules.

## Architecture
- **Language**: [DETECT_ON_SETUP]
- **Framework**: [DETECT_ON_SETUP]
- **Package Manager**: [DETECT_ON_SETUP]
- **Test Framework**: [DETECT_ON_SETUP]
- **Database**: [DETECT_ON_SETUP]
- **Deployment**: [DETECT_ON_SETUP]

## Key Commands
- `[pkg] run dev` — Start dev server
- `[pkg] run build` — Production build
- `[pkg] run test` — Run tests
- `[pkg] run lint` — Lint check
- `[pkg] run type-check` — Type checking

## Code Standards
- TypeScript strict mode when applicable
- No `any` types — use `unknown` or proper typing
- All exports must have JSDoc/docstrings
- Max file length: 300 lines — split if larger
- Prefer composition over inheritance
- Error handling: never swallow errors silently
- All API endpoints must have input validation
- Environment variables: never hardcode secrets
- Use meaningful variable names — no single-letter vars except loop counters

## Workflow Rules
- For ANY task >50 lines of code: Enter Plan Mode FIRST (Shift+Tab)
- For ANY commit: All tests must pass (enforced by Stop hook)
- For ANY architecture decision: Use the CTO agent for evaluation
- For ANY security-related code: Mandatory security-auditor agent review
- After ANY correction from user: Update this file under "Self-Correction Rules"
- Use subagents for parallel work when tasks are independent
- Use background tasks (Ctrl+B) for long-running processes
- Use Playwright MCP for visual/UI verification when available
- ALWAYS "prove it works" before marking any task done

## Background Task Rules
- ALWAYS run dev servers in background
- ALWAYS run watch processes in background
- Run security scans in background while developing
- Use /tasks to monitor running background agents

## Memory Management
- After each session, run /wrap-up to consolidate learnings
- Update this file with key architectural decisions
- Memory priority: Security rules > Architecture > Code style > Preferences
- When Claude makes a mistake and is corrected, add rule to Self-Correction section

## Agent Team Patterns
Use agent teams for large, parallelizable work:

| Scenario | Team Composition | When to Use |
|----------|-----------------|-------------|
| New Feature | architect + qa-engineer + security-auditor + performance-engineer | Multi-file feature development |
| Code Review | security-auditor + performance-engineer + code-reviewer | Before any merge/deploy |
| Debugging | 3-5 agents with different hypotheses (any combination) | Hard-to-find bugs |
| Refactoring | architect + code-reviewer + doc-writer | Major restructuring |
| Data Pipeline | data-scientist + devops-engineer + qa-engineer | Data-intensive features |

## The Ultimate Development Cycle
```
1. PLAN   → Plan Mode + CTO Agent → Design the solution
2. BUILD  → Act Mode + Agents → Build (parallel when possible)
3. TEST   → QA Agent + Background Tasks → Tests run while you review
4. REVIEW → /review-with-fresh-eyes → Clean-context review
5. SHIP   → /ship → All quality gates must pass
6. LEARN  → /wrap-up → Update this file with learnings
```

## The Self-Healing Loop
```
Code Change → PostToolUse Hook (type check on .ts/.tsx)
           → Stop Hook (build verification)
           → Pre-Commit (all tests must pass)
           → If fail: Claude auto-fixes (up to 3 cycles)
           → If still fail: Alert developer
           → After human fix: /learn-mistake → Rule added here
           → Next time: Hook prevents the mistake entirely
```

## Diagnostic & Power Commands
Built-in commands for troubleshooting and monitoring:
- `/doctor` — Health check on Claude Code installation
- `/debug` — Troubleshoot current session issues
- `/stats` — Session statistics (press `r` to cycle 7d/30d/all-time)
- `/compact [instructions]` — Manual context compaction (e.g., `/compact preserve all function signatures`)
- `/teleport` — Transfer session between claude.ai web and local terminal
- `/add-dir <path>` — Add working directories mid-session (monorepo support)
- `/config` — Settings interface with search functionality
- `/tasks` — Monitor running background agents

## Monorepo & Multi-Directory Support
When working across multiple directories:
- Use `claude --add-dir ../other-project` to include additional directories
- CLAUDE.md files from additional directories are loaded automatically (env: `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`)
- Each directory can have its own `.claude/agents/` and `.claude/commands/`

## Session Management
- `/teleport` — Move sessions between claude.ai and terminal (requires same auth)
- Auto-compact triggers at ~85% context capacity (tuned via `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`)
- Session memory auto-records and recalls important decisions across sessions
- Use `/compact preserve [topic]` to protect critical context during compaction

## Environment Variables Reference
Key env vars configured in `.claude/settings.json`:
| Variable | Value | Purpose |
|----------|-------|---------|
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` | `1` | Enable multi-agent teams |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liortesta/ClawdAgent](https://github.com/liortesta/ClawdAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
