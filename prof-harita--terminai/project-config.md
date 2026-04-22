---
trigger: always_on
description: > **Purpose**: This document is the single source of truth for AI agents working
---

# AGENTS.md — The Definitive Guide for AI Agents

> **Purpose**: This document is the single source of truth for AI agents working
> on TerminAI. Read this completely before making any changes.
>
> **Last Updated**: January 21, 2026  
> **Scope**: All packages, all workflows, all agents

---

## Table of Contents

1. [Project Identity](#project-identity)
2. [Architecture Overview](#architecture-overview)
3. [The Golden Rules](#the-golden-rules)
4. [Package Guide](#package-guide)
5. [Development Workflows](#development-workflows)
6. [Code Standards](#code-standards)
7. [Safety & Governance](#safety--governance)
8. [Testing Protocol](#testing-protocol)
9. [Documentation Guidelines](#documentation-guidelines)
10. [Upstream Maintenance](#upstream-maintenance)
11. [Common Pitfalls](#common-pitfalls)
12. [Quick Reference](#quick-reference)

---

## Project Identity

### What is TerminAI?

TerminAI is an **AI-powered system operator** — not just a coding assistant.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         TERMINAI THESIS                              │
│                                                                     │
│   "Google provides the intelligence.                                │
│    TerminAI provides the root access and the guardrails."          │
│                                                                     │
│   We are building GOVERNED AUTONOMY for systems and servers.       │
└─────────────────────────────────────────────────────────────────────┘
```

### What Makes Us Different

| Capability                                           | TerminAI | Others  |
| ---------------------------------------------------- | -------- | ------- |
| Actually executes (not just suggests)                | ✅       | Rare    |
| Policy gating (approval before action)               | ✅       | ❌      |
| Three-axis security (Outcome/Intent/Domain)          | ✅       | ❌      |
| Audit trail (immutable, non-disableable)             | ✅       | ❌      |
| Voice control (push-to-talk STT/TTS)                 | ✅       | ❌      |
| Agent-to-Agent protocol                              | ✅       | ❌      |
| Multi-LLM (Gemini, ChatGPT OAuth, OpenAI-compatible) | ✅       | Limited |
| Native Windows support                               | ✅       | Limited |
| Recipes engine (governed playbooks)                  | ✅       | ❌      |

### The Vision Stack

```
┌─────────────────────────────────────────────────────────────────────┐
│  END USERS: "Fix my computer" — actually works                     │
├─────────────────────────────────────────────────────────────────────┤
│  POWER USERS: Voice control, process orchestration, MCP extensions │
├─────────────────────────────────────────────────────────────────────┤
│  DEVELOPERS: A2A protocol, PTY bridge, policy engine primitives    │
├─────────────────────────────────────────────────────────────────────┤
│  ORGANIZATIONS: Non-repudiable logs, approval workflows, fleet-ready│
└─────────────────────────────────────────────────────────────────────┘
```

---

## Architecture Overview

### Monorepo Structure

```
terminaI/
├── packages/
│   ├── core/           # 🧠 Engine: tools, policy, safety, telemetry
│   ├── cli/            # ⌨️  Terminal UI (Ink/React) + RuntimeManager
│   ├── desktop/        # 🖥️  Tauri app + PTY bridge
│   ├── a2a-server/     # 🔌 Agent-to-Agent control plane
│   ├── termai/         # 🚀 The `terminai` launcher
│   ├── microvm/        # 🔐 Micro-VM runtime (Firecracker/Hyper-V/Virt.fw)
│   ├── sandbox-image/  # 📦 Docker/container image + T-APTS Python package
│   ├── evolution-lab/  # 🧪 Automated testing harness (Docker-default)
│   ├── cloud-relay/    # ☁️  Cloud relay server
│   ├── test-utils/     # 🧰 Testing utilities
│   ├── desktop-linux-atspi-sidecar/   # 🐧 Linux GUI sidecar
│   ├── desktop-windows-driver/        # 🪟 Windows automation driver
│   ├── vscode-ide-companion/          # 💻 VS Code integration
│   ├── web-client/     # 🌐 Web client
│   └── api/            # 📡 API definitions
├── docs/               # 📚 Upstream documentation
├── docs-terminai/      # 📖 TerminAI-specific documentation
├── .agent/workflows/   # 🔄 Agent workflow definitions
├── schemas/            # 📐 JSON Schemas (settings, policy)
└── scripts/            # ⚙️  Build and utility scripts
```

### Data Flow

```mermaid
flowchart TB
    subgraph Input["User Input"]
        CLI[CLI Terminal]
        Desktop[Desktop App]
        A2A[A2A Clients]
        Voice[Voice/PTT]
    end

    subgraph Core["@terminai/core"]
        Policy[Policy Engine]
        Approval[Approval Ladder]
        Tools[Tool Scheduler]
        Brain[Thinking Orchestrator]
        Recipes[Recipes Engine]
    end

    subgraph LLM["LLM Providers"]
        Gemini[Gemini API]
        ChatGPT[ChatGPT OAuth]
        OpenAI[OpenAI-Compatible]
    end

    subgraph Execution["Execution Layer"]
        Shell[Shell Tool]
        FileOps[File Operations]
        REPL[REPL Tool]
        GUI[GUI Automation]
        Computer[Computer Session Manager]
    end

    subgraph Safety["Safety Layer"]
        Audit[Audit Ledger]
        Sandbox[Sandbox Controller]
        MCP[MCP OAuth Provider]
    end

    Input --> Policy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Prof-Harita/terminaI](https://github.com/Prof-Harita/terminaI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
