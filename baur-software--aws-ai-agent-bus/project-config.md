---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🚨 CRITICAL: Test Debt Tracking

**NEVER IGNORE SKIPPED TESTS** - They represent production bugs waiting to happen.

Current test debt status:

- **9 skipped tests** in dashboard-ui (see `dashboard-ui/TODO_TESTS.md`)
- **1 timer logic bug** - could cause data loss in production
- **9 component integration gaps** - no coverage for critical UI flows

Before claiming "production ready": ALL tests must pass without `.skip()`.

## Core Commands

### MCP Server Setup

```bash
# Configure Claude Code to use the Rust MCP server
# This enables Claude Code to directly use MCP tools while developing!
# Add to Claude Code settings or mcp_servers.json:
{
  "agent-mesh": {
    "command": "use_aws_mcp",
    "env": {
      "AWS_REGION": "us-west-2"
    }
  }
}

# Claude Code can then use tools like:
# - mcp__aws__kv_get/set for KV storage
# - mcp__aws__artifacts_get/put for file storage
# - mcp__aws__events_send for EventBridge
# - mcp__aws__workflow_start for Step Functions
# - mcp__aws__agent_* for agent operations
```

### Development

```bash
# MCP Server development (Rust implementation)
cd mcp-rust
cargo build                 # Build Rust MCP server
cargo test                  # Run full test suite
cargo run                   # Start MCP server on stdio (default for MCP clients)
cargo install --path .     # Install use_aws_mcp binary globally

# Dashboard UI development (SolidJS)
cd dashboard-ui
npm install
npm run dev              # Start Vite dev server on port 5173+
npm run build            # Build for production
npm run preview          # Preview production build

# Environment Configuration
cp .env.example .env     # Create local environment file
# Edit .env to set VITE_MCP_SERVER_URL for custom MCP server location

# Run both together (from root)
npm run dev:all          # Starts both MCP server and dashboard UI

# Google Analytics reports
npm run setup:ga-google-cloud    # Complete Google Cloud + GA setup assistant
npm run setup:ga-credentials     # Interactive GA credentials setup
npm run report:users-by-country  # Live GA report (requires credentials)
npm run report:users-by-country-sample  # Sample data report

# Infrastructure (Terraform)
# Set AWS profile (recommended - avoids hardcoding in Terraform files)
export AWS_PROFILE=baursoftware
export AWS_REGION=us-west-2

npm run tf:fmt            # Format Terraform files
npm run tf:init           # Initialize workspace (set WS and ENV vars)
npm run tf:plan           # Plan infrastructure changes
npm run tf:apply          # Apply infrastructure changes
npm run tf:destroy        # Destroy infrastructure

# Alternative: Use backend config file (profile centralized)
cd infra/workspaces/small/kv_store
terraform init -backend-config=backend.hcl

# Event Monitoring Infrastructure (PowerShell on Windows)
cd infra/workspaces/small/events_monitoring
powershell -ExecutionPolicy Bypass -File deploy.ps1   # Plan changes
powershell -ExecutionPolicy Bypass -File apply.ps1    # Apply changes
```

### Environment Variables

#### Infrastructure & AWS

```bash
# Required for infrastructure operations
export WS=small/kv_store    # Workspace path
export ENV=dev              # Environment (dev/staging/prod)

# AWS Configuration  
export AWS_REGION=us-west-2
export AGENT_MESH_KV_TABLE=agent-mesh-kv
export AGENT_MESH_ARTIFACTS_BUCKET=agent-mesh-artifacts
export AGENT_MESH_EVENT_BUS=agent-mesh-events
```

#### Dashboard UI Configuration (dashboard-ui/.env)

```bash
# MCP Server Configuration
VITE_MCP_SERVER_URL=http://localhost:3001    # MCP server URL for proxy
VITE_MCP_SERVER_PORT=3001                    # MCP server port
VITE_DEV_MODE=true                           # Enable development features

# Optional overrides
VITE_APP_TITLE="Custom Dashboard Title"
VITE_API_ENDPOINT=http://localhost:3001/api
```

## Architecture Overview

### Core Components

- **MCP Server** (`mcp-rust/`): Rust-based Model Context Protocol server providing AI assistants with AWS service access
- **Dashboard Server** (`dashboard-server/`): WebSocket API gateway for the SolidJS dashboard UI
- **Dashboard UI** (`dashboard-ui/`): SolidJS-based frontend for workflow management and real-time monitoring
- **Infrastructure** (`infra/`): Terraform modules organized into small/medium/large workspaces
- **Agent System** (`.claude/`): Sophisticated agent orchestration with conductors, critics, and specialists

### MCP Server Structure (Rust)

- `src/`: Rust implementation of MCP server with AWS integrations
- Features: KV storage, artifacts, events, workflows, analytics
- Performance-optimized with async/await and tokio runtime
- Type-safe AWS SDK integration

### AI Chat System

- **AWS Bedrock Integration**: Real Claude AI via Bedrock Runtime API
- **Streaming Support**: Token-by-token responses for better UX
- **Multi-Model**: Claude 3.5 Sonnet (default), Haiku, Opus
- **MCP Tool Integration**: Automatic analytics and data tool usage
- **Conversation Memory**: Full context maintained per session
- **Cost Tracking**: Real token usage metrics from Bedrock

See: [AWS Bedrock Chat Setup Guide](docs/aws-bedrock-chat-setup.md)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Baur-Software/aws-ai-agent-bus](https://github.com/Baur-Software/aws-ai-agent-bus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
