---
trigger: always_on
description: 1. ALL operations MUST be concurrent/parallel in a single message
---

# Claude Code Configuration - SPARC Development Environment

## 🚨 CRITICAL: CONCURRENT EXECUTION & FILE MANAGEMENT

**ABSOLUTE RULES**:
1. ALL operations MUST be concurrent/parallel in a single message
2. **NEVER save working files, text/mds and tests to the root folder**
3. ALWAYS organize files in appropriate subdirectories

### ⚡ GOLDEN RULE: "1 MESSAGE = ALL RELATED OPERATIONS"

**MANDATORY PATTERNS:**
- **TodoWrite**: ALWAYS batch ALL todos in ONE call (5-10+ todos minimum)
- **Task tool**: ALWAYS spawn ALL agents in ONE message with full instructions
- **File operations**: ALWAYS batch ALL reads/writes/edits in ONE message
- **Bash commands**: ALWAYS batch ALL terminal operations in ONE message
- **Memory operations**: ALWAYS batch ALL memory store/retrieve in ONE message

### 📁 File Organization Rules

**NEVER save to root folder. Use these directories:**
- `/src` - Source code files
- `/tests` - Test files
- `/docs` - Documentation and markdown files
- `/config` - Configuration files
- `/scripts` - Utility scripts
- `/examples` - Example code

## Project Overview

This project uses SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) methodology with Claude-Flow orchestration for systematic Test-Driven Development.

## SPARC Commands

### Core Commands
- `npx claude-flow sparc modes` - List available modes
- `npx claude-flow sparc run <mode> "<task>"` - Execute specific mode
- `npx claude-flow sparc tdd "<feature>"` - Run complete TDD workflow
- `npx claude-flow sparc info <mode>` - Get mode details

### Batchtools Commands
- `npx claude-flow sparc batch <modes> "<task>"` - Parallel execution
- `npx claude-flow sparc pipeline "<task>"` - Full pipeline processing
- `npx claude-flow sparc concurrent <mode> "<tasks-file>"` - Multi-task processing

### Build Commands
- `npm run build` - Build project
- `npm run test` - Run tests
- `npm run lint` - Linting
- `npm run typecheck` - Type checking

## SPARC Workflow Phases

1. **Specification** - Requirements analysis (`sparc run spec-pseudocode`)
2. **Pseudocode** - Algorithm design (`sparc run spec-pseudocode`)
3. **Architecture** - System design (`sparc run architect`)
4. **Refinement** - TDD implementation (`sparc tdd`)
5. **Completion** - Integration (`sparc run integration`)

## Code Style & Best Practices

- **Modular Design**: Files under 500 lines
- **Environment Safety**: Never hardcode secrets
- **Test-First**: Write tests before implementation
- **Clean Architecture**: Separate concerns
- **Documentation**: Keep updated

### 🚨 AI TypeScript Error Prevention Guide

**Why AI-generated TypeScript throws overwhelming errors:**

1. **Version mismatches** - Claude may generate code assuming different TypeScript, Node.js, or framework versions
2. **Missing or wrong type definitions** - AI often imports libraries without installing their `@types/*` packages
3. **Over-typed or fantasy types** - AI sometimes "hallucinates" overly strict types or entire interfaces
4. **Project config mismatch** - Strict tsconfig.json settings can make small type mismatches balloon into hundreds of errors
5. **Copy-paste fragmentation** - Inconsistent types and imports when code is pasted in parts

**How to fix or reduce the frustration:**

- **Check versions first**: Run `npx tsc --version`, `node -v`, `npm list typescript`
- **Simplify types**: For prototyping, loosen strict settings in tsconfig.json (set `strict: false`)
- **Install missing types**: `npm install --save-dev @types/node @types/express`
- **Regenerate in context**: Paste package.json and tsconfig.json into Claude before asking for code
- **Iterate smaller**: Generate one file/module at a time instead of entire web apps

### 📋 TypeScript Development Checklist

**Before generating TypeScript code:**
1. ✅ Check exact TypeScript version: `npx tsc --version`
2. ✅ Verify Node.js version: `node -v`
3. ✅ List installed types: `npm list @types`
4. ✅ Review tsconfig.json strict settings
5. ✅ Identify required dependencies and their @types packages

**During development:**
1. ✅ Generate one module at a time
2. ✅ Test compilation after each file: `npx tsc --noEmit`
3. ✅ Install types immediately when importing new libraries
4. ✅ Use interface over type for extensible definitions
5. ✅ Avoid fantasy types - stick to what actually exists

**After code generation:**
1. ✅ Run type checking: `npm run typecheck`
2. ✅ Run linting: `npm run lint`
3. ✅ Test with actual data/API responses
4. ✅ Verify imports resolve correctly

## 🚀 Available Agents (54 Total)

### Core Development
`coder`, `reviewer`, `tester`, `planner`, `researcher`

### Swarm Coordination
`hierarchical-coordinator`, `mesh-coordinator`, `adaptive-coordinator`, `collective-intelligence-coordinator`, `swarm-memory-manager`

### Consensus & Distributed
`byzantine-coordinator`, `raft-manager`, `gossip-coordinator`, `consensus-builder`, `crdt-synchronizer`, `quorum-manager`, `security-manager`

### Performance & Optimization
`perf-analyzer`, `performance-benchmarker`, `task-orchestrator`, `memory-coordinator`, `smart-agent`

### GitHub & Repository
`github-modes`, `pr-manager`, `code-review-swarm`, `issue-tracker`, `release-manager`, `workflow-automation`, `project-board-sync`, `repo-architect`, `multi-repo-swarm`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dakgmen/trades-helper-mvp](https://github.com/dakgmen/trades-helper-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
