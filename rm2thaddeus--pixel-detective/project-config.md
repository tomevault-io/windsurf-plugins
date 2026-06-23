---
trigger: always_on
description: Use this file for finding rules in project
---

# Cross-Reference Guide - Comprehensive Rule Navigation

## 🗺️ RULE ECOSYSTEM OVERVIEW

Your `.cursor` rules are organized into **3 tiers** with **cross-cutting concerns**:

```
🏢 PROJECT-WIDE (.cursor/rules/)
├── 🔥 sprint-lessons-learned.mdc         # Master reference - Sprint 10 failures
├── ⚡ quick-troubleshooting-index.mdc    # Emergency rapid resolution  
├── 🧪 sprint10-testing-patterns.mdc     # Prevention through testing
├── 🔧 mcp-browser-tools-setup.mdc       # 3-component MCP protocol
├── 📋 sprint-planning.mdc               # Sprint workflow & planning
├── 🌐 use-mcp-servers.mdc               # MCP server integration
├── 🐛 debugging.mdc                     # General debugging protocols
├── 💬 commentsoverwrite.mdc             # Documentation standards
└── 📖 cross-reference-guide.mdc         # This navigation guide

🎨 FRONTEND (frontend/.cursor/rules/)
├── 📋 frontend-development-index.mdc    # Master index - START HERE for frontend work
├── 💧 nextjs-hydration-prevention.mdc   # Server/client consistency
├── 🔄 react-query-api-integration.mdc   # Server state management
├── 🏗️ component-architecture-patterns.mdc # Component design & composition patterns
├── 🎨 ux-workflow-patterns.mdc          # User experience & workflow patterns
└── ⚡ nextjs-performance-optimization.mdc # Next.js performance optimization

⚙️ BACKEND (backend/.cursor/rules/)
├── 📋 backend-development-index.mdc     # Master index - START HERE for backend work
├── 🔗 fastapi-dependency-injection.mdc  # Circular import prevention
├── 🏗️ fastapi-microservice-patterns.mdc # Complete service architecture patterns
├── 🤖 ml-service-integration.mdc         # GPU-optimized ML service patterns
└── 🔌 api-design-patterns.mdc           # RESTful API design & data modeling
```

## 🎯 SCENARIO-BASED NAVIGATION

### 🚨 "I Have an Emergency Issue"
**START HERE:** `.cursor/rules/quick-troubleshooting-index.mdc`
- **30-second fixes** for common Sprint 10 problems
- **Diagnostic flowchart** to route you to detailed solutions
- **Severity levels** to prioritize fixes

**Then go to specific rule:**
- Hydration errors → `frontend/.cursor/rules/nextjs-hydration-prevention.mdc`
- Import errors → `backend/.cursor/rules/fastapi-dependency-injection.mdc`
- MCP failures → `.cursor/rules/mcp-browser-tools-setup.mdc`

### 🛠️ "I'm Starting New Development Work"
**1. Check Sprint Context:** `.cursor/rules/sprint-lessons-learned.mdc`
   - Review applicable lessons from Sprint 10 failures
   - Check mandatory pre-work checklists

**2. Choose Development Track:**
   - **Frontend work (any)** → `frontend/.cursor/rules/frontend-development-index.mdc` (comprehensive guide)
   - **Frontend components** → `frontend/.cursor/rules/component-architecture-patterns.mdc`
   - **Frontend UX** → `frontend/.cursor/rules/ux-workflow-patterns.mdc`
   - **Frontend performance** → `frontend/.cursor/rules/nextjs-performance-optimization.mdc`
   - **Hydration issues** → `frontend/.cursor/rules/nextjs-hydration-prevention.mdc`
   - **Backend work (any)** → `backend/.cursor/rules/backend-development-index.mdc` (comprehensive guide)
   - **Backend API work** → `backend/.cursor/rules/api-design-patterns.mdc`
   - **Backend ML work** → `backend/.cursor/rules/ml-service-integration.mdc`
   - **Backend services** → `backend/.cursor/rules/fastapi-microservice-patterns.mdc`
   - **Import issues** → `backend/.cursor/rules/fastapi-dependency-injection.mdc`
   - **MCP integration** → `.cursor/rules/mcp-browser-tools-setup.mdc`

**3. Set Up Testing:** `.cursor/rules/sprint10-testing-patterns.mdc`
   - Pre-commit hooks for your development area
   - Continuous validation patterns

### 🧪 "I Want to Prevent Issues Through Testing"
**START HERE:** `.cursor/rules/sprint10-testing-patterns.mdc`
- **Component-level tests** for hydration safety
- **Import validation** for backend architecture
- **MCP integration tests** for tool reliability
- **CI/CD workflows** for continuous prevention

**Related rules:**
- Testing patterns reference specific prevention rules
- Each test maps to a Sprint 10 lesson learned

### 📋 "I'm Planning a Sprint"
**START HERE:** `.cursor/rules/sprint-planning.mdc`
- Sprint workflow integration with MCP tools
- PRD generation and documentation standards

**Integration points:**
- Health check: `.cursor/rules/sprint10-testing-patterns.mdc`
- MCP setup: `.cursor/rules/mcp-browser-tools-setup.mdc`
- Lessons: `.cursor/rules/sprint-lessons-learned.mdc`

### 🔧 "I Need to Set Up MCP Tools"
**START HERE:** `.cursor/rules/mcp-browser-tools-setup.mdc`
- 3-component verification protocol
- Troubleshooting for common setup issues

**Integration with:**
- Testing: `.cursor/rules/sprint10-testing-patterns.mdc` (MCP testing section)
- Sprint work: `.cursor/rules/sprint-planning.mdc` (MCP-driven workflow)

## 📊 RULE DEPENDENCY MAP

### Core Dependencies (Must Read First)
```
sprint-lessons-learned.mdc (MASTER reference)
    ├── Referenced by: quick-troubleshooting-index.mdc
    ├── Referenced by: sprint10-testing-patterns.mdc
    ├── Referenced by: nextjs-hydration-prevention.mdc
    ├── Referenced by: react-query-api-integration.mdc
    └── Referenced by: fastapi-dependency-injection.mdc
```

### Technical Implementation Chain
```
1. sprint-lessons-learned.mdc (What went wrong?)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rm2thaddeus/Pixel_Detective](https://github.com/rm2thaddeus/Pixel_Detective) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
