---
trigger: always_on
description: > **Universal AI Agent Instructions**: This file follows the [AGENTS.md standard](https://agents.md/) and works with Claude Code, GitHub Copilot, Cursor, Google Jules, OpenAI Codex, Aider, and 20+ other AI coding assistants.
---

# Exocortex Development - AI Agent Coordination Hub

> **Universal AI Agent Instructions**: This file follows the [AGENTS.md standard](https://agents.md/) and works with Claude Code, GitHub Copilot, Cursor, Google Jules, OpenAI Codex, Aider, and 20+ other AI coding assistants.

## 🎯 Project Context: AI-Driven Knowledge Management

**What is Exocortex?**

Exocortex is a **knowledge management system** that gives users convenient control over all their knowledge. Started as an Obsidian plugin for ontology-driven layouts (Areas → Projects → Tasks), it has evolved into a larger system with CLI capabilities and advanced semantic features.

**Core Philosophy**: AI-driven development

- This project is developed **exclusively by AI agents**
- Each session runs **parallel and independent** of which agent is used
- **Continuous self-improvement** of AI instructions based on learned experience

**Product Capabilities**:

- Renders ontology-driven layouts inside Obsidian
- Links hierarchical knowledge structures (Areas → Projects → Tasks)
- Tracks effort history and work state transitions
- Surfaces vote-based prioritization signals
- CLI tools for automation (`packages/cli`)
- Shared semantic utilities (`packages/core`)

**Architecture**: Clean Architecture with strict layering

- `packages/obsidian-plugin/src/presentation` - UI components and renderers
- `packages/obsidian-plugin/src/application` - Use cases and orchestration
- `packages/obsidian-plugin/src/domain` - Pure business logic (framework-independent)
- `packages/obsidian-plugin/src/infrastructure` - I/O, external dependencies, Obsidian API
- `packages/core` - Shared utilities across all packages
- `packages/cli` - Command-line interface tools

---

## 🚨 RULE #1 (MOST CRITICAL): WORKTREES ONLY

**⚠️ THIS IS THE MOST IMPORTANT RULE - VIOLATION IS UNACCEPTABLE ⚠️**

**The `exocortex/` directory is STRICTLY READ-ONLY.**

ALL code changes MUST happen through git worktrees in the `worktrees/` subdirectory.

### Why This Rule Exists

1. **Parallel AI agent work**: Multiple agents work simultaneously without conflicts
2. **Safe experimentation**: Each worktree is isolated sandbox
3. **Clean coordination**: Git worktrees show active work across all agents
4. **Prevents corruption**: Main repository stays pristine

### Enforcement

**❌ ABSOLUTELY FORBIDDEN:**

```bash
cd ~/Developer/exocortex-development/exocortex
vim src/some-file.ts              # ❌ NEVER DO THIS!
git commit -am "changes"          # ❌ BLOCKED!
```

**✅ ONLY CORRECT WAY:**

```bash
# 1. Create worktree
cd ~/Developer/exocortex-development/exocortex
git worktree add ../worktrees/exocortex-[agent]-[type]-[task] -b feature/[task]

# 2. Work in worktree
cd ../worktrees/exocortex-[agent]-[type]-[task]
vim src/some-file.ts              # ✅ CORRECT!
git commit -am "feat: changes"    # ✅ SAFE!
```

### Validation Before Starting Work

**ALWAYS verify your location:**

```bash
pwd
# MUST output: .../exocortex-development/worktrees/exocortex-*
# If "worktrees/" is missing → STOP IMMEDIATELY!
```

---

## 🚨 RULE #2 (SECOND MOST CRITICAL): MANDATORY SELF-IMPROVEMENT

**⚠️ EVERY COMPLETED TASK MUST PRODUCE POST-MORTEM WITH IMPROVEMENT PROPOSALS ⚠️**

This project evolves through **iterative self-improvement** of AI agent instructions. Your experience is valuable data for future agents.

### Post-Mortem Report (MANDATORY)

After EVERY completed task, you MUST:

1. **Document errors encountered** - Every error, no matter how small
2. **Describe solutions applied** - Exact steps that fixed each error
3. **Extract lessons learned** - Patterns, insights, gotchas discovered
4. **Propose documentation improvements** - Specific additions to AGENTS.md, CLAUDE.md, etc.
5. **WAIT FOR USER APPROVAL** - Present report to user, get explicit permission before editing any files

### ⚠️ CRITICAL: DO NOT AUTO-EDIT DOCUMENTATION

**You MUST NOT edit AGENTS.md, CLAUDE.md, or any instruction files without explicit user permission.**

**Correct workflow**:

1. ✅ Write post-mortem report
2. ✅ Propose improvements with exact text to add
3. ✅ **PRESENT to user and ASK for permission**
4. ✅ **WAIT for user approval**
5. ✅ **ONLY THEN** edit documentation files

**Forbidden**:

- ❌ Automatically editing instruction files after task completion
- ❌ Updating documentation "based on learnings" without asking
- ❌ Committing changes to AGENTS.md, CLAUDE.md without permission

### Post-Mortem Template

```markdown
## Task: [Feature/Fix Name]

### Completed

- [What was implemented]
- [Tests added: X unit + Y E2E]
- [Coverage: Z%]

### Errors Encountered & Solutions

1. **[Error Category]**: [Error description]
   - **Error**: [Exact error message / stack trace]
   - **Root Cause**: [Why it happened]
   - **Solution**: [Exact steps to fix]
   - **Prevention**: [How to avoid in future]

2. **[Next Error]**: ...

### Lessons Learned

- **Pattern discovered**: [New pattern found in codebase]
- **Gotcha identified**: [Unexpected behavior or edge case]
- **Best practice**: [Better way to do X]
- **Tool insight**: [How to use tool Y more effectively]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitelev/exocortex](https://github.com/kitelev/exocortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
