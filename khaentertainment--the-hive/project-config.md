---
trigger: always_on
description: **Multi-Model AI Orchestration Platform - GitHub Public Repository**
---

# Claude Code Configuration - The Hive Live Repository

**Multi-Model AI Orchestration Platform - GitHub Public Repository**

*Version: 3.0 | Live Repo | Last Updated: August 10, 2025*

## 🏗️ Repository Structure

This is the **LIVE GITHUB REPOSITORY** containing the public-facing installer, distribution, and production code.

### **Repository Purpose:**
- **Public GitHub Repo**: https://github.com/KHAEntertainment/the-hive
- **Installer Distribution**: Main entry point for users to install The Hive
- **Production Code**: Stable, tested code ready for public use
- **Documentation**: User guides, installation instructions, API docs

### **Cross-Workspace Coordination:**
- Development and research happens in `../thehive-research/`
- Production code and releases managed HERE
- **Unified Session Data**: All session data symlinked to research workspace
- **Shared Memory**: `.swarm/` and `.superclaude/` directories are symlinked
- SuperClaude sessions work seamlessly from either workspace location

---

## 🚨 CRITICAL: PUBLIC REPOSITORY GUIDELINES

**ABSOLUTE RULES**:
1. NO sensitive information (API keys, credentials, internal docs)
2. ALL commits must be production-ready and tested
3. Follow semantic versioning for all releases
4. Maintain clean commit history for public visibility

### ⚡ GOLDEN RULE: "PUBLIC-READY ONLY"

**MANDATORY PATTERNS:**
- **Security First**: Never commit secrets or sensitive configuration
- **User-Focused**: All content serves end users, not internal development
- **Quality Assurance**: Every commit passes full test suite
- **Documentation**: Clear, comprehensive user documentation

### 📁 Public Repository Structure

```
the-hive/
├── 📄 README.md                  # Main project documentation
├── 🚀 install.sh                 # Primary installer script
├── 📦 package.json              # Node.js dependencies
├── 🔧 enhancements/             # Enhancement scripts
│   └── scripts/
│       ├── sc-git-mcp.sh        # Git-MCP integration
│       ├── superclaude-init.sh  # SuperClaude setup
│       └── dashboard.sh         # Dashboard management
├── 📚 docs/                     # User documentation
│   ├── installation/            # Installation guides
│   ├── usage/                   # Usage examples
│   └── api/                     # API documentation
├── 🧪 tests/                    # Public test suites
└── 🏷️ releases/                 # Release artifacts
```

---

## Project Overview

The Hive is a multi-model AI orchestration platform that enables collective intelligence through sophisticated coordination of multiple AI models. This repository provides the installer and production distribution.

## 🚀 Installation & Setup

### Quick Install
```bash
# Install The Hive platform
curl -fsSL https://raw.githubusercontent.com/KHAEntertainment/the-hive/main/install.sh | bash

# Or download and run locally
git clone https://github.com/KHAEntertainment/the-hive.git
cd the-hive
chmod +x install.sh
./install.sh
```

### Prerequisites
- Claude Code environment
- Node.js 18+ 
- Git
- Unix-like system (macOS, Linux, WSL)

### Enhanced Installation
```bash
# Install with SuperClaude enhancements
./enhancements/scripts/superclaude-init.sh

# Add Git-MCP integration
./enhancements/scripts/sc-git-mcp.sh --setup

# Enable dashboard
./enhancements/scripts/dashboard.sh --enable
```

## 🎯 Features

### Core Capabilities
- **Multi-Model Coordination**: Orchestrate multiple AI models simultaneously
- **Byzantine Consensus**: Fault-tolerant distributed decision making
- **Persistent Memory**: Cross-session knowledge graphs
- **Git-MCP Integration**: Convert GitHub repos to MCP data sources
- **Real-time Dashboard**: Monitor agent coordination and performance

### Supported AI Models
- **Horizon-Beta**: Architecture and complex reasoning
- **GLM-4.5-air**: Documentation and content generation
- **GPT-OSS-20b**: Analysis and validation
- **Gemini**: Research and cross-validation
- **Claude**: Coordination and orchestration

## 📋 Quick Start Examples

### Basic Collective Intelligence
```bash
# Start hive-mind session
/sc:hive-mind "Analyze best practices for API design" --consensus --agents 5

# Research coordination
/sc:research-swarm "Modern authentication patterns 2025" --strategy topic_division

# Individual model access
/sc:openrouter "Design scalable microservices architecture" --model openrouter/horizon-beta
```

### Git-MCP Repository Integration
```bash
# Add repository data sources
/sc:git-mcp facebook/react
/sc:git-mcp anthropics/claude-code

# Interactive setup
/sc:git-mcp
```

### Cross-Workspace Development
```bash
# Sessions work from either workspace (unified via symlinks)
cd ../thehive-research && /sc:hive-mind "research new features"
cd ../the-hive && /sc:swarm "update production documentation"

# Session data is shared - resume from anywhere
/sc:resume sc-swarm-20250809-001136  # Works from either location
```

## 🔧 Configuration

### Environment Variables
```bash
export HIVE_WORKSPACE="live-repo"
export HIVE_MEMORY_NAMESPACE="superclaude/thehive/live"
export HIVE_SESSION_TYPE="production"
```

### MCP Server Setup
```bash
# Add Claude-Flow MCP server
claude mcp add claude-flow npx claude-flow@alpha mcp start

# Verify installation
claude mcp list
npx claude-flow@alpha swarm status
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KHAEntertainment/the-hive](https://github.com/KHAEntertainment/the-hive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
