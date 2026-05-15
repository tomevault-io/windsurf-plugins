---
trigger: always_on
description: **Project:** mcp-finnhub - Finnhub MCP Server
---

# CLAUDE.md - AI Agent Operational Guide

**Project:** mcp-finnhub - Finnhub MCP Server
**Language:** Python 3.11+
**MCP Server:** Serena
**Project Name in Serena:** mcp-finnhub

---

## 🚀 Quick Start for New Sessions

If you're starting a new session or lost context, follow these steps:

### 1. Activate Serena MCP for this project

```
Use Serena MCP tool: activate_project
Project path: /Users/robsherman/Servers/mcp-finnhub
```

### 2. Read the current state

```
Read Serena memory: todo       # What you're currently working on
Read Serena memory: progress   # What's been completed
Read Serena memory: phases     # Overall project roadmap
```

### 3. Check the CHANGELOG

```
Read: CHANGELOG.md             # Git commits and version history
```

### 4. Review planning documents (if needed)

```
Read: docs/ARCHITECTURE.md     # Complete architecture
Read: docs/PATTERNS.md         # MCP server patterns learned
Read: docs/DEVELOPMENT.md      # Dev workflow, testing, tools
```

---

## 📂 Project Structure

```
mcp-finnhub/
├── CLAUDE.md                  # ← YOU ARE HERE - Operational guide
├── CHANGELOG.md               # Version history with semantic versioning
├── README.md                  # User-facing documentation (TBD)
├── .mcp.json                  # Local MCP server configuration
├── .mcp.json.README.md        # Configuration guide
│
├── docs/                      # 📚 PLANNING DOCUMENTATION
│   ├── ARCHITECTURE.md        # Complete 23-tool architecture
│   ├── PATTERNS.md            # Learned patterns from mcp-fred, alpha-vantage, snowflake
│   └── DEVELOPMENT.md         # Dev workflow, Ruff, PyTest, 80% coverage
│
├── .serena/                   # 🧠 SERENA MCP OPERATIONAL STATE
│   ├── project.yml            # Serena project config
│   └── memories/
│       ├── phases.md          # 7 phases, 550 SP, 17 sprints
│       ├── todo.md            # Current sprint tasks
│       └── progress.md        # Completed work history
│
├── src/mcp_finnhub/           # Source code (to be built)
├── tests/                     # Test suite (to be built)
├── pyproject.toml             # Python project config (to be built)
└── .env.example               # Environment variables template (to be built)
```

---

## 🧠 Serena Memory Guide

### **phases** - Project Roadmap

**Purpose:** High-level project plan broken into 7 phases

**Read with:** `Read Serena memory: phases`

**Contains:**
- Phase 1: Foundation & Core Infrastructure (90 SP)
- Phase 2: API Client & Job Management (90 SP)
- Phase 3: Core Tools - Mandatory (100 SP)
- Phase 4: Stock Analysis Tools (90 SP)
- Phase 5: Multi-Asset & Discovery Tools (90 SP)
- Phase 6: Management Tools & Integration (50 SP)
- Phase 7: Documentation & Release (40 SP)

**Total:** 550 story points, 17 sprints

**When to read:** When you need to understand the overall roadmap or plan the next phase

---

### **todo** - Current Sprint

**Purpose:** Active sprint tasks with story breakdown

**Read with:** `Read Serena memory: todo`

**Contains:**
- Current sprint number and story points
- All stories in the sprint with tasks
- Task checkboxes for tracking completion
- Definition of done for each story
- Sprint completion criteria
- Next sprint preview

**When to read:**
- ⭐ **START OF EVERY SESSION** - This tells you what you're working on
- When resuming work after interruption
- To check what's left in current sprint

**Example structure:**
```
Sprint 1.1 - Project Scaffold (30 SP)

Story 1.1.1 (8 SP): Create project structure
- [ ] Task 1
- [ ] Task 2
Status: Not Started

Story 1.1.2 (8 SP): Setup pyproject.toml
- [ ] Task 1
Status: Not Started
```

---

### **progress** - Completed Work

**Purpose:** History of what's been completed

**Read with:** `Read Serena memory: progress`

**Contains:**
- Completed phases and sprints
- Key milestones achieved
- Git commits summary
- Important decisions made
- Current phase/sprint status
- Velocity tracking

**When to read:**
- When you need to understand what's already done
- Before starting a new phase
- To avoid duplicating work
- To understand project history

**Update frequency:** After each sprint completion

---

## 📋 Development Workflow

### Starting a Sprint

1. **Read current state:**
   ```
   Read Serena memory: todo
   Read Serena memory: progress
   ```

2. **Execute sprint:**
   - Work through all stories in order
   - No mid-sprint updates to user
   - Update task checkboxes in `todo` memory as you complete them
   - Make git commits with semantic versioning

3. **Complete sprint:**
   - Provide comprehensive summary to user
   - Update `progress` memory with sprint completion
   - Update `todo` memory with next sprint
   - Update CHANGELOG.md

### Making Changes

1. **Git commits:**
   ```
   Format: <type>(<scope>): <subject>
   Types: feat, fix, docs, style, refactor, test, chore, perf

   Examples:
   feat(tools): add technical analysis tool with 4 operations
   fix(api): handle rate limit errors correctly
   docs: update ARCHITECTURE.md with testing strategy
   test(utils): add tests for token estimator
   ```

2. **Update CHANGELOG.md:**
   - Add entry under [Unreleased] section
   - Use semantic versioning
   - Group by Added/Changed/Fixed/Removed

3. **Update Serena memories:**
   - Check off tasks in `todo` as completed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cfdude/mcp-finnhub](https://github.com/cfdude/mcp-finnhub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
