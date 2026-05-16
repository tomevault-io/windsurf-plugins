---
trigger: always_on
description: description: UNDERSTAND SYMindX Cursor rules hierarchy when working with project rules
---


---
description: UNDERSTAND SYMindX Cursor rules hierarchy when working with project rules
globs: .cursor/rules/*.mdc
---

# Cursor Rules Framework for SYMindX

This document outlines the comprehensive rule framework used in the SYMindX project, providing intelligent navigation and context-aware rule activation for AI agents working within Cursor IDE.

## Framework Overview

The SYMindX Cursor rules framework consists of **24 comprehensive rules** (000-index.mdc + 000-022 core rules) organized in a hierarchical, extensible structure with intelligent cross-referencing and automated verification tools.

### Complete Rule Index

| Rule | Purpose | Dependencies | Triggers |
|------|---------|--------------|----------|
| 000-index.mdc | Master navigation hub | All rules | Always active |
| 000-rules.mdc | Meta-rule defining framework standards | None | *.mdc files |
| 001-symindx-workspace.mdc | Core project architecture | None | Always active |
| 002-cursor-rules-framework.mdc | Framework organization | 000-rules.mdc | *.mdc files |
| 003-typescript-standards.mdc | TypeScript and Bun standards | 001-symindx-workspace.mdc | **/*.ts,**/*.js |
| 004-architecture-patterns.mdc | Modular design patterns | 003-typescript-standards.mdc | Core development |
| 005-ai-integration-patterns.mdc | AI portal integration | 004-architecture-patterns.mdc | AI provider work |
| 006-web-interface-patterns.mdc | React and frontend | 003-typescript-standards.mdc | **/*.tsx,website/** |
| 007-extension-system-patterns.mdc | Platform integrations | 004-architecture-patterns.mdc | Extensions work |
| 008-testing-and-quality-standards.mdc | Testing strategies | All development rules | **/*.test.ts,**/*.spec.ts |
| 009-deployment-and-operations.mdc | Docker and deployment | 001-symindx-workspace.mdc | Docker files |
| 010-security-and-authentication.mdc | Security standards | All system rules | Security-related code |
| 011-data-management-patterns.mdc | Database and memory | 003-typescript-standards.mdc | Memory/database work |
| 012-performance-optimization.mdc | Performance patterns | All development rules | Performance-critical code |
| 013-error-handling-logging.mdc | Error handling patterns | All development rules | Error-prone code |
| 014-cli-and-tooling-patterns.mdc | CLI design patterns | 003-typescript-standards.mdc | CLI development |
| 015-configuration-management.mdc | Config and environment | 010-security-and-authentication.mdc | Config files |
| 016-documentation-standards.mdc | Documentation patterns | None | **/*.md,docs/** |
| 017-community-and-governance.mdc | Open source practices | 016-documentation-standards.mdc | Community files |
| 018-git-hooks.mdc | Git automation | 008-testing-and-quality-standards.mdc | Git operations |
| 019-background-agents.mdc | Cloud-powered agents | 004-architecture-patterns.mdc | Background tasks |
| 020-mcp-integration.mdc | Model Context Protocol | 005-ai-integration-patterns.mdc | MCP work |
| 021-advanced-context.mdc | Context-aware activation | All rules | Dynamic contexts |
| 022-workflow-automation.mdc | Workflow orchestration | All automation rules | Workflow files |

## Documentation and Tools Integration

### Core Documentation (@.cursor/docs/)

The framework integrates closely with comprehensive documentation that agents should reference:

- **@.cursor/docs/quick-start.md** - Developer onboarding and first-time setup
- **@.cursor/docs/architecture.md** - Detailed system architecture documentation  
- **@.cursor/docs/contributing.md** - Development workflow and contribution guidelines

These docs provide essential context for:
- New developer onboarding
- System architecture understanding
- Development workflow guidance
- Code contribution standards

### Development Tools (@.cursor/tools/)

The framework leverages specialized development tools that agents can utilize:

- **@.cursor/tools/project-analyzer.md** - Project structure analysis and code metrics
- **@.cursor/tools/debugging-guide.md** - Comprehensive debugging strategies and troubleshooting
- **@.cursor/tools/code-generator.md** - Component templates and code generation patterns
- **@.cursor/tools/verify-rule-links.js** - Automated cross-reference verification tool

These tools enable:
- Automated project analysis and insights
- Systematic debugging approaches
- Consistent code generation
- Framework quality assurance

## Cross-Reference System

### Reference Syntax
```markdown
@rule-name.mdc           # Direct rule reference
@.cursor/docs/file.md    # Documentation reference
@.cursor/tools/file.md   # Development tool reference
```

### Smart Navigation by Context

**🚀 Starting New Features**
```
Foundation: @001-symindx-workspace.mdc → @003-typescript-standards.mdc
Architecture: @004-architecture-patterns.mdc
Component-Specific: @005-ai-integration-patterns.mdc OR @007-extension-system-patterns.mdc
Quality: @008-testing-and-quality-standards.mdc
Documentation: @.cursor/docs/architecture.md, @.cursor/tools/code-generator.md
```

**🔧 Daily Development Tasks**
```
Standards: @003-typescript-standards.mdc
Performance: @012-performance-optimization.mdc  
Error Handling: @013-error-handling-logging.mdc
Tools: @.cursor/tools/debugging-guide.md, @.cursor/tools/project-analyzer.md
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
