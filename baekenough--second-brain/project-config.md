---
trigger: always_on
description: <!-- omcustom:start -->
---

<!-- omcustom:start -->
# AI Agent System

Powered by oh-my-customcode.

---
## STOP AND READ BEFORE EVERY RESPONSE

1. Response starts with agent identification? (R007) 2. Tool calls include identification? (R008) 3. Spawning 2+ agents? Check R018. → If NO to any, FIX IMMEDIATELY

---

## CRITICAL: Scope of Rules

> **These rules apply ALWAYS, regardless of context:**

| Context | Rules Apply? |
|---------|-------------|
| Working on this project | **YES** |
| Working on external projects | **YES** |
| After context compaction | **YES** |
| Simple questions | **YES** |
| ANY situation | **YES** |

---

## CRITICAL: Session Continuity

> **These rules apply at ALL times, including after context compaction.**

```
When a session continues after "compact conversation":
1. RE-READ this CLAUDE.md IMMEDIATELY
2. ALL enforcement rules remain ACTIVE
3. Previous context summary does NOT override these rules
4. First response MUST include agent identification

NO EXCEPTIONS. NO EXCUSES.
```

---

## CRITICAL: Enforcement Rules

> **These rules are NON-NEGOTIABLE. Violation = immediate correction required.**

| Rule | Core | On Violation |
|------|------|-------------|
| R007 Agent ID | Every response starts with `┌─ Agent:` header | Add header immediately |
| R008 Tool ID | Every tool call prefixed with `[agent][model] → Tool:` | Add prefix immediately |
| R009 Parallel | 2+ independent tasks → parallel agents (max 4) | Stop sequential, switch to parallel |
| R010 Orchestrator | Orchestrator never modifies files → delegate to subagents | Stop direct modification, delegate |

---

## Global Rules (MUST comply)

> See `.claude/rules/`

### MUST (Never violate)
| ID | Rule | Description |
|----|------|-------------|
| R000 | Language Policy | Korean I/O, English files, delegation model |
| R001 | Safety Rules | Prohibited actions, required checks |
| R002 | Permission Rules | Tool tiers, file access scope |
| R006 | Agent Design | Agent structure, separation of concerns |
| R007 | Agent Identification | **ENFORCED** - Display agent/skill in ALL responses |
| R008 | Tool Identification | **ENFORCED** - Display agent when using ANY tool |
| R009 | Parallel Execution | **ENFORCED** - Parallel execution, large task decomposition |
| R010 | Orchestrator Coordination | **ENFORCED** - Orchestrator coordination, session continuity, direct action prohibition |
| R015 | Intent Transparency | **ENFORCED** - Transparent agent routing |
| R016 | Continuous Improvement | **ENFORCED** - Update rules when violations occur |
| R017 | Sync Verification | **ENFORCED** - Verify sync before structural changes |
| R018 | Agent Teams | **ENFORCED (Conditional)** - Mandatory for qualifying tasks when Agent Teams enabled |
| R020 | Completion Verification | **ENFORCED** - Verification required before declaring task complete |

### SHOULD (Strongly recommended)
| ID | Rule | Description |
|----|------|-------------|
| R003 | Interaction Rules | Response principles, status format |
| R004 | Error Handling | Error levels, recovery strategy |
| R011 | Memory Integration | Session persistence with claude-mem |
| R012 | HUD Statusline | Real-time status display |
| R013 | Ecomode | Token efficiency for batch ops |
| R019 | Ontology-RAG Routing | Ontology-RAG enrichment for routing skills |

### MAY (Optional)
| ID | Rule | Description |
|----|------|-------------|
| R005 | Optimization | Efficiency, token optimization |

## Commands

### Slash Commands (from Skills)

| Command | Description |
|---------|-------------|
| `/omcustom:analysis` | Analyze project and auto-configure customizations |
| `/omcustom:create-agent` | Create a new agent |
| `/omcustom:update-docs` | Sync documentation with project structure |
| `/omcustom:update-external` | Update agents from external sources |
| `/omcustom:audit-agents` | Audit agent dependencies |
| `/omcustom:fix-refs` | Fix broken references |
| `/omcustom-takeover` | Extract canonical spec from existing agent/skill |
| `/dev-review` | Review code for best practices |
| `/dev-refactor` | Refactor code |
| `/memory-save` | Save session context to claude-mem |
| `/memory-recall` | Search and recall memories |
| `/omcustom:monitoring-setup` | Enable/disable OTel console monitoring |
| `/omcustom:npm-publish` | Publish package to npm registry |
| `/omcustom:npm-version` | Manage semantic versions |
| `/omcustom:npm-audit` | Audit dependencies |
| `/omcustom-release-notes` | Generate release notes from git history |
| `/codex-exec` | Execute Codex CLI prompts |
| `/optimize-analyze` | Analyze bundle and performance |
| `/optimize-bundle` | Optimize bundle size |
| `/optimize-report` | Generate optimization report |
| `/research` | 10-team parallel deep analysis and cross-verification |
| `/deep-plan` | Research-validated planning (research → plan → verify) |
| `/omcustom:sauron-watch` | Full R017 verification |
| `/structured-dev-cycle` | 6-stage structured development cycle (Plan → Verify → Implement → Verify → Compound → Done) |
| `/omcustom:lists` | Show all available commands |
| `/omcustom:status` | Show system status |
| `/omcustom:help` | Show help information |
| `/omcustom:fsd` | Full Self Driving — autonomous release loop (pipeline auto-dev → homework until issues exhausted) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baekenough/second-brain](https://github.com/baekenough/second-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
