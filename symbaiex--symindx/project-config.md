---
trigger: always_on
description: This master index provides intelligent navigation through the comprehensive SYMindX Cursor rules framework, documentation, and development tools. Use this as your primary entry point for context-aware rule selection and cross-reference navigation.
---

# SYMindX Cursor Rules Master Index

This master index provides intelligent navigation through the comprehensive SYMindX Cursor rules framework, documentation, and development tools. Use this as your primary entry point for context-aware rule selection and cross-reference navigation.

## Quick Navigation by Development Context

### 🚀 Starting New Feature Development
```
🎯 Project Context: @001-symindx-workspace.mdc
📋 Architecture Guide: @.cursor/docs/architecture.md
🔧 TypeScript Standards: @003-typescript-standards.mdc
🏗️ Architecture Patterns: @004-architecture-patterns.mdc
📝 Code Generation: @.cursor/tools/code-generator.md
```

### 🔧 Daily Development Tasks
```
⚡ Performance: @012-performance-optimization.mdc
🛡️ Security: @010-security-and-authentication.mdc
🐛 Error Handling: @013-error-handling-logging.mdc
🔍 Debugging: @.cursor/tools/debugging-guide.md
📊 Analysis: @.cursor/tools/project-analyzer.md
```

### 🧠 Component Development
```
🤖 AI Portals: @005-ai-integration-patterns.mdc + @.cursor/tools/code-generator.md
💾 Memory Systems: @011-data-management-patterns.mdc + @.cursor/docs/architecture.md
🔌 Extensions: @007-extension-system-patterns.mdc + @.cursor/docs/contributing.md
🌐 Web Interface: @006-web-interface-patterns.mdc
⌨️ CLI Tools: @014-cli-and-tooling-patterns.mdc
```

### 🤖 Automation & Integration
```
📜 Git Hooks: @018-git-hooks.mdc
🔄 Background Agents: @019-background-agents.mdc
🔗 MCP Integration: @020-mcp-integration.mdc
🎭 Context Awareness: @021-advanced-context.mdc
🎼 Workflow Orchestration: @022-workflow-automation.mdc
```

### 🚀 Deployment & Operations
```
🐳 Docker & Deployment: @009-deployment-and-operations.mdc
⚙️ Configuration: @015-configuration-management.mdc
🧪 Testing: @008-testing-and-quality-standards.mdc
📚 Documentation: @016-documentation-standards.mdc
```

## Rule Dependencies and Prerequisites

### Foundation Layer (Start Here)
Essential understanding required for all development:

```mermaid
graph TD
    A[@001-symindx-workspace.mdc] --> B[@003-typescript-standards.mdc]
    A --> C[@.cursor/docs/architecture.md]
    A --> D[@.cursor/docs/quick-start.md]
    B --> E[@004-architecture-patterns.mdc]
    C --> E
```

**Priority Order:**
1. **@001-symindx-workspace.mdc** - Project architecture and standards
2. **@.cursor/docs/architecture.md** - Detailed system architecture
3. **@003-typescript-standards.mdc** - Language and runtime standards
4. **@004-architecture-patterns.mdc** - Design patterns and modularity

### Development Workflow Layer
Build upon foundation with development standards:

```mermaid
graph TD
    A[@004-architecture-patterns.mdc] --> B[@008-testing-and-quality-standards.mdc]
    A --> C[@013-error-handling-logging.mdc]
    A --> D[@015-configuration-management.mdc]
    B --> E[@.cursor/tools/project-analyzer.md]
    C --> F[@.cursor/tools/debugging-guide.md]
```

**Core Development Rules:**
- **@008-testing-and-quality-standards.mdc** - Testing methodologies
- **@013-error-handling-logging.mdc** - Error patterns and logging
- **@015-configuration-management.mdc** - Configuration and secrets
- **@.cursor/tools/debugging-guide.md** - Debugging strategies
- **@.cursor/tools/project-analyzer.md** - Code analysis tools

### Component Specialization Layer
Choose based on your component type:

#### AI Portal Development
```mermaid
graph TD
    A[@005-ai-integration-patterns.mdc] --> B[@012-performance-optimization.mdc]
    A --> C[@010-security-and-authentication.mdc]
    A --> D[@.cursor/tools/code-generator.md]
    B --> E[Portal Templates]
    C --> E
    D --> E
```

#### Memory System Development
```mermaid
graph TD
    A[@011-data-management-patterns.mdc] --> B[@012-performance-optimization.mdc]
    A --> C[@010-security-and-authentication.mdc]
    A --> D[@.cursor/docs/architecture.md]
    B --> E[Memory Implementation]
    C --> E
    D --> E
```

#### Platform Extension Development
```mermaid
graph TD
    A[@007-extension-system-patterns.mdc] --> B[@010-security-and-authentication.mdc]
    A --> C[@015-configuration-management.mdc]
    A --> D[@.cursor/docs/contributing.md]
    B --> E[Extension Implementation]
    C --> E
    D --> E
```

### Operations and Quality Layer
Production readiness and maintenance:

```mermaid
graph TD
    A[Component Development] --> B[@009-deployment-and-operations.mdc]
    A --> C[@016-documentation-standards.mdc]
    A --> D[@017-community-and-governance.mdc]
    B --> E[Production Ready]
    C --> E
    D --> E
```

## Smart Rule Selection Guide

### Context-Based Selection Matrix

| Development Context | Primary Rule | Supporting Rules | Tools & Docs |
|---------------------|--------------|------------------|--------------|
| **New AI Portal** | @005-ai-integration-patterns.mdc | @012-performance-optimization.mdc, @010-security-and-authentication.mdc | @.cursor/tools/code-generator.md |
| **Memory Issues** | @011-data-management-patterns.mdc | @012-performance-optimization.mdc, @013-error-handling-logging.mdc | @.cursor/tools/debugging-guide.md |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
