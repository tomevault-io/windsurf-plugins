---
trigger: always_on
description: 🚨 SYNCHRONIZED FROM: AGENTS.md (MASTER SOURCE)
---

# [PROJECT_NAME] AI Instructions (GitHub Copilot Enhanced)

<!--
    🚨 SYNCHRONIZED FROM: AGENTS.md (MASTER SOURCE)

    This file is automatically synchronized across all AI platforms.
    DO NOT edit directly - edit the master file and run sync:
    .\tools\ai-instructions-sync.ps1 -Action sync-all
    
    AI PLATFORM FILES THAT MUST BE SYNCHRONIZED:
    - AGENTS.md (MASTER SOURCE - ROOT LEVEL)
    - .github/copilot-instructions.md (THIS FILE - GitHub Copilot)
    - CLAUDE.md (Claude AI platform - ROOT LEVEL)
    - GEMINI.md (Google Gemini AI platform - ROOT LEVEL)
    - .vscode/README.md (VS Code)
    - .zed/README.md (Zed Editor)
    
    MULTI-OS SUPPORT REQUIREMENTS:
    - Windows: PowerShell + Docker Desktop
    - macOS/Linux: Bash + Docker Desktop
    - WSL2: /mnt/e/HPZ620/adx-core-k/ (RECOMMENDED)
    
    🚨 PATH STRUCTURE RULES:
    - ALL paths use forward slashes: /
    - ALL files start from project root: [PROJECT_NAME]/
    - Kiro specs go in: .kiro/specs/[project]/
    - AI agents go in: ai-agents/[agent].md
    - Kiro hooks go in: .kiro/hooks/[hook-name].kiro.hook
    - Steering documents go in: .kiro/steering/[decision-name].md
    - Platform files are in their respective directories
    
    NEVER edit platform files directly - always edit the master file (AGENTS.md)!
-->

This file contains GitHub Copilot-specific AI instructions optimized for Copilot's code generation capabilities, IntelliSense integration, and enterprise features with FAANG-level enhancements.

**🔄 AUTOMATIC SYNCHRONIZATION ACTIVE**
- ✅ This file is synchronized from the master source (AGENTS.md)
- ⚡ Changes to master automatically sync to all AI platforms  
- 🛡️ Automatic backup protection before every sync
- 📊 Comprehensive validation ensures consistency
- 🎯 Edit AGENTS.md → All platforms updated automatically

**🚀 GitHub Copilot Enhanced Features**
- **Code Generation**: Advanced code completion and generation with context awareness
- **Enterprise Integration**: Seamless integration with GitHub Enterprise and organizational policies
- **Multi-language Support**: Intelligent suggestions across multiple programming languages
- **Real-time Collaboration**: Team-aware suggestions based on repository patterns
- **Security-first**: Built-in security scanning and vulnerability prevention

## Core Directives (FAANG-Enhanced)

### 🎯 **Primary Rules (Google SRE-Inspired)**
- **IDE Tools First**: Use built-in diagnostics, language servers, integrated terminals before external calls
- **UTC+7 Timezone**: All datetime operations use Asia/Ho_Chi_Minh timezone
- **Real Time Only**: Never fake timestamps, always use actual current time
- **Multi-Platform Sync**: Update all AI instruction files simultaneously across all OS
- **Cross-OS Compatibility**: Support Windows, macOS, Linux, WSL2 seamlessly
- **Metrics-Driven Development**: Track AI effectiveness and optimization opportunities
- **Self-Healing Environment**: Automatic detection and resolution of common issues

### 📁 **CRITICAL PATH RULES (Enhanced)**
- **ALWAYS use relative paths**: All paths are relative to project root (no `[PROJECT_NAME]/` prefix)
- **Kiro Specs Location**: `.kiro/specs/[project-name]/` for all specifications
- **Kiro Hooks Location**: `.kiro/hooks/[hook-name].kiro.hook` for automation hooks
- **Steering Documents**: `.kiro/steering/[decision-name].md` for ADRs and governance
- **AI Agent Files**: `ai-agents/[agent-name].md` for specialized agents
- **AI Work Sessions**: `ai-agents-works/sessions/` with single-file approach using status-based naming
- **Session Naming**: `{STATUS}-{timestamp}-{session-name}.md` format (e.g., ACTIVE-20250113-142530-fixing-auth-service.md)
- **No Deep Nesting**: Maximum 3 levels deep for new files
- **Consistent Separators**: Use forward slashes `/` in all path references
- **Context-Aware Creation**: Use intelligent agent selector before file creation
- **Feature-Specific Specs**: `.kiro/specs/[feature-name]/` for individual feature specifications

### 🚨 **SIMPLIFIED WORK LOSS PREVENTION PROTOCOL (Netflix-Inspired Self-Healing)**
- **Start session IMMEDIATELY**: Use `.\ai-agents-works\automation\ai-work-tracker.ps1 -Action start-session -AgentName "[agent]" -Objective "[task]" -Priority "[priority]"`
- **Single file per session**: Each session = one markdown file with status-based naming: `{STATUS}-{timestamp}-{session-name}.md`
- **Status management**: ACTIVE → COMPLETED/PAUSED/CANCELLED with automatic file renaming
- **Update frequently**: Use `update-session` action every 15-30 minutes with progress and token usage
- **Self-contained sessions**: All work context preserved in single comprehensive markdown file
- **Pattern Learning**: Document successful approaches in session knowledge sections

### 🏗️ **Architecture Context (Meta-Style Intelligence)**
- **Temporal-First**: Complex operations = Temporal workflows (not CRUD)
- **Dual-Mode Services**: HTTP server + Temporal worker modes
- **Multi-Tenant**: Database/application/workflow isolation with tenant context
- **Micro-Frontend**: Module Federation with domain boundaries
- **Rust Backend**: 11 microservices + shared utilities
- **React Frontend**: TypeScript micro-frontends + shared packages

## Intelligent Agent Selection (Meta-Inspired)

### **Smart Agent Routing Engine**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hscale/ai-instructions-template](https://github.com/hscale/ai-instructions-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
