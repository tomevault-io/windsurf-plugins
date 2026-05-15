---
trigger: always_on
description: This project uses the Enhanced BMAD Method - combining persona-based AI agents with advanced context management, persistent memory, and comprehensive testing orchestration.
---

# Enhanced BMAD Method with Context Management

This project uses the Enhanced BMAD Method - combining persona-based AI agents with advanced context management, persistent memory, and comprehensive testing orchestration.

## 🚀 Quick Start

### ✨ NEW: Autonomous Orchestration (Recommended)
```bash
# Simply tell the orchestrator what you want - it handles EVERYTHING automatically
"I want to create a task management app"

# The orchestrator will AUTONOMOUSLY:
# 1. Create a complete workflow plan
# 2. Delegate to Mary (analyst) for project brief
# 3. Continue to John (PM) for PRD
# 4. Proceed to Winston (architect) for technical design
# 5. Validate with Sarah (PO)
# 6. Create stories with Bob (SM)
# 7. Implement with James (Dev)
# 8. Review with Quinn (QA)
# 9. Test thoroughly
# 10. Report back with complete results

# You just wait for the magic to happen! ✨
```

### Manual Agent Approach (Advanced Users)
```bash
# If you prefer direct control, you can still call agents manually
"Use bmad-analyst to research and create project brief for [your project]"
"Use bmad-pm to create PRD based on the project brief"
"Use bmad-architect to create technical architecture"
```

### For Existing Projects
```bash
# Initialize BMAD + Context system
"Set up this codebase with BMAD method"
# Orchestrator will delegate to project-initializer automatically
```

## 🤖 Agent System Overview

### 🎯 Autonomous Orchestration System
The bmad-orchestrator is now your SINGLE POINT OF INTERACTION:
- **Automatic Workflow Management** - Just describe what you want, orchestrator handles the rest
- **Intelligent Agent Delegation** - Knows which agent to use and when
- **Progress Tracking** - Updates you as work progresses
- **Quality Gates** - Ensures each phase meets standards before proceeding
- **Error Recovery** - Handles issues and retries automatically
- **Health Monitoring** - Tracks agent performance and creates new agents as needed

### Core BMAD Agents (Personas)
- **bmad-orchestrator** - Autonomous master coordinator with context management
- **bmad-analyst** (Mary) - Business Analyst for research and ideation
- **bmad-pm** (John) - Product Manager for PRDs and user stories
- **bmad-architect** (Winston) - System Architect for technical design
- **bmad-po** (Sarah) - Product Owner for validation and process
- **bmad-sm** (Bob) - Scrum Master for detailed story creation
- **bmad-dev** (James) - Developer for implementation
- **bmad-qa** (Quinn) - QA Engineer for code review
- **bmad-ux** (Sally) - UX Designer for interfaces
- **bmad-devops** (Alex) - DevOps Engineer for infrastructure
- **bmad-tech-writer** (Morgan) - Technical Writer for documentation

### Infrastructure Agents
- **testing-orchestrator** - Comprehensive testing coordination
- **project-initializer** - Project setup and configuration

## 📋 Automatic Agent Selection

When responding to user requests, the appropriate BMAD agent is automatically selected:

### Planning & Analysis Tasks
- **Market research, competitive analysis, brainstorming** → `bmad-analyst` (Mary)
- **Creating PRDs, epics, user stories** → `bmad-pm` (John)
- **System design, architecture decisions** → `bmad-architect` (Winston)
- **Document validation, sharding, process integrity** → `bmad-po` (Sarah)

### Development Tasks
- **Creating detailed development stories** → `bmad-sm` (Bob)
- **Code implementation, debugging, testing** → `bmad-dev` (James)
- **Code review, quality assurance** → `bmad-qa` (Quinn)

### Quality Assurance
- **Comprehensive testing orchestration** → `testing-orchestrator`
- **Code quality review** → `bmad-qa` (Quinn)

### Supporting Tasks
- **UI/UX design, wireframes, mockups** → `bmad-ux` (Sally)
- **CI/CD pipelines, deployment setup** → `bmad-devops` (Alex)
- **API documentation, user guides** → `bmad-tech-writer` (Morgan)

### Coordination & Setup
- **Workflow guidance, agent selection help** → `bmad-orchestrator`
- **Project initialization** → `project-initializer`

## 🔄 Enhanced Workflow

### 🚨 IMPORTANT: Execution Mode
All agents now operate in **EXECUTION MODE** - they create actual files instead of just describing what they would do. Every agent uses Write/Edit tools to create real deliverables.

### Phase 1: Planning (Web UI)
1. **Research & Ideation** → bmad-analyst uses Write tool to create project brief at `projectdocs/{project}-brief.md`
2. **Product Definition** → bmad-pm uses Write tool to create PRD at `projectdocs/{project}-prd.md`
3. **Architecture Design** → bmad-architect uses Write tool to create architecture at `projectdocs/{project}-architecture.md`
4. **Validation** → bmad-po uses Write tool to create validation report at `projectdocs/{project}-validation.md`

### Phase 2: Development (IDE)
1. **Story Preparation** → bmad-sm uses Write tool to create stories at `projectdocs/{project}-stories.md`
2. **Implementation** → bmad-dev uses Write/Edit tools to create actual code files
3. **Quality Review** → bmad-qa uses Write tool to create review at `projectdocs/{project}-qa-report.md`
4. **Testing** → testing-orchestrator uses Write tool to create test results

### Phase 3: Deployment
1. **Infrastructure** → bmad-devops sets up CI/CD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wolverin0/bmad-claude-agents](https://github.com/wolverin0/bmad-claude-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
