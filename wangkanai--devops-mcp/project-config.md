---
trigger: always_on
description: Comprehensive context for Claude Code to work effectively with the Azure DevOps MCP server for seamless project management and development workflows.
---

# CLAUDE.md - DevOps MCP for Claude Code Integration

Comprehensive context for Claude Code to work effectively with the Azure DevOps MCP server for seamless project management and development workflows.

---

## 📋 **Project Overview**

### **devops-mcp** - Azure DevOps MCP Server for Claude
**Technology**: Node.js, TypeScript, MCP Protocol  
**Purpose**: Dynamic Azure DevOps integration with intelligent directory-based authentication switching  
**Status**: ✅ **PRODUCTION READY** - Active deployment with >95% test coverage  

**GitHub**: <https://github.com/wangkanai/devops-mcp>  
**NPM Package**: [@wangkanai/devops-mcp](https://www.npmjs.com/package/@wangkanai/devops-mcp)  
**Features**: Local `.azure-devops.json` configuration, secure PAT tokens, comprehensive testing, full Azure DevOps API integration

---

## 🚀 **Quick Start for Claude Users**

### **Immediate Setup (2 minutes)**

```bash
# 1. Install via Claude Code (Recommended)
claude mcp add devops-mcp -- -y @wangkanai/devops-mcp

# 2. Create configuration in your project directory
cat > .azure-devops.json << EOF
{
  "organizationUrl": "https://dev.azure.com/your-org",
  "project": "YourProject", 
  "pat": "your-pat-token-here",
  "description": "Azure DevOps configuration for this repository"
}
EOF

# 3. Secure your configuration
echo ".azure-devops.json" >> .gitignore

# 4. Verify installation
mcp__devops-mcp__get-current-context
```

### **Claude Desktop Users**

Add to your MCP settings file:

```json
{
  "mcpServers": {
    "devops-mcp": {
      "command": "npx",
      "args": ["-y", "@wangkanai/devops-mcp"]
    }
  }
}
```

**Settings Location**:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

---

## 🎯 **Core Capabilities for Claude**

### **Smart Context Switching**
Claude automatically detects project context based on your current directory:
- **RiverSync projects**: Auto-switches to `https://dev.azure.com/riversync`
- **Mula projects**: Auto-switches to `https://dev.azure.com/mula-x`
- **Any project**: Uses local `.azure-devops.json` configuration

### **Complete Azure DevOps Integration**
- ✅ **Work Item Management**: Create, update, query work items with full hierarchy support
- ✅ **Repository Operations**: Access repositories, pull requests, branch information
- ✅ **Build & Pipeline**: Trigger pipelines, monitor builds, check deployment status
- ✅ **Project Context**: Get current configuration, validate authentication

---

## 🛠️ **Available MCP Commands**

### **Work Item Commands**
| Command | Purpose | Key Parameters |
|---------|---------|----------------|
| `mcp__devops-mcp__get-work-items` | Query work items | `--wiql`, `--ids`, `--fields` |
| `mcp__devops-mcp__create-work-item` | Create new work items | `--type`, `--title`, `--parent`, `--assignedTo` |
| `mcp__devops-mcp__update-work-item` | Update existing items | `--id`, `--state`, `--assignedTo`, `--parent` |
| `mcp__devops-mcp__add-work-item-comment` | Add comments | `--id`, `--comment` |

### **Repository Commands**
| Command | Purpose | Key Parameters |
|---------|---------|----------------|
| `mcp__devops-mcp__get-repositories` | List repositories | `--includeLinks` |
| `mcp__devops-mcp__get-pull-requests` | Get PR information | `--status`, `--repositoryId`, `--createdBy` |

### **Build & Pipeline Commands**
| Command | Purpose | Key Parameters |
|---------|---------|----------------|
| `mcp__devops-mcp__get-builds` | Get build history | `--definitionIds`, `--top` |
| `mcp__devops-mcp__trigger-pipeline` | Start builds | `--definitionId`, `--definitionName`, `--sourceBranch` |
| `mcp__devops-mcp__get-pipeline-status` | Check build status | `--buildId`, `--definitionId`, `--includeTimeline` |

### **Context Commands**
| Command | Purpose | Key Parameters |
|---------|---------|----------------|
| `mcp__devops-mcp__get-current-context` | Show configuration | `--directory` |

---

## 💡 **Claude-Optimized Workflows**

### **🆕 Feature Development Workflow**

```bash
# Create complete feature hierarchy
# 1. Epic for major initiative
mcp__devops-mcp__create-work-item \
  --type "Epic" \
  --title "User Authentication System" \
  --description "Implement comprehensive user authentication and authorization"

# 2. Feature under Epic (assuming Epic ID = 1100)
mcp__devops-mcp__create-work-item \
  --type "Feature" \
  --title "OAuth 2.0 Integration" \
  --parent 1100 \
  --iterationPath "MyProject\\Sprint 1"

# 3. User Stories under Feature (assuming Feature ID = 1101)
mcp__devops-mcp__create-work-item \
  --type "User Story" \
  --title "As a user, I want to login with Google OAuth" \
  --parent 1101 \
  --assignedTo "frontend@company.com"

# 4. Implementation Tasks (assuming User Story ID = 1102)
mcp__devops-mcp__create-work-item \
  --type "Task" \
  --title "Design OAuth login UI components" \
  --parent 1102 \
  --assignedTo "frontend@company.com"

mcp__devops-mcp__create-work-item \
  --type "Task" \
  --title "Implement OAuth backend service" \
  --parent 1102 \
  --assignedTo "backend@company.com"
```

### **🐛 Bug Triage and Resolution**

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangkanai/devops-mcp](https://github.com/wangkanai/devops-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
