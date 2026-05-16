---
trigger: always_on
description: **The Definitive Reference for Framework Development**
---

# 🏢 The System - ASDO Framework (Autonomous Software Development Organization)

**The Definitive Reference for Framework Development**

---

## Framework Overview

**The System** is an agentic framework that simulates a complete software development organization. It orchestrates 26 specialized AI agents across 6 departments to take ideas from concept to production, with you as the founder making key decisions at human-in-the-loop (HITL) gates.

### Current Framework Status
- **Agents:** 26
- **Commands:** 59
- **Stages:** 5 (Architecture → Product → Development → Release → Go Live)
- **HITL Gates:** 8
- **Status:** Production-ready framework with comprehensive documentation and help system

---

## Company Structure

```
👤 Human Founder (You)
     │
     ▼
🎩 Founder-Advisor
     │
     ├─────────┬─────────┬─────────┬─────────┬─────────┬─────────┐
     ▼         ▼         ▼         ▼         ▼         ▼         ▼
📐 Architecture🎨 Design 📦 Product💻 Development🚀 Release 🌐 Go Live
   Department   Department Department  Department   Department (Optional)
     │         │         │         │              │         │
     │         ├── 🔍 API├── 👔 Product├── 👨‍💼 Principal├── 📝 Writer├── 🚀 SRE Deploy
     │         │   Discovery │    Lead │    Developer │        │
     │         ├── 🎨 Style├── 📅 Planner├── 🧪 QA      ├── 🔐 Security├── 🛡️ SRE Ops
     │         │   Manager│         │    Engineer  │    Engineer │
     │         ├── 📋 Docs├── 💼 Analyst├── 🗄️ DB Dev  ├── 📦 Release│
     │         │   Auditor│         │              │    Engineer │
     │         ├── 🖼️ Wireframe│         ├── ⚙️ BE Dev  └── 🚀 DevOps │
     │         │   Generator│         │                          │
     │         ├── 🏛️ Enterprise│         ├── 🎨 FE Dev              │
     │         │   Architect│         │                          │
     │         ├── 🔍 UX   │         └── 🔗 Integration         │
     │         │   Analyzer│         │                          │
     │         └── 🚀 Prototype│         └── 🐛 Bug Fixer (Utility)│
     │           Developer │                                    │
     └── 🏗️ Solution       │                                    │
         Architect         └────────────────────────────────────┘
```

---

## Departments & Agents (26 Total)

### 🎩 Founder-Advisor
Your chief of staff. All communication flows through this agent.

### 📐 Architecture Department (Stage 1)
- **🏗️ Solution Architect** - System design, technical architecture, ADRs, Project Explorer (legacy analysis)

### 🎨 Design Department (Stage 1.5) ✨ **ENHANCED**
- **🔍 API Discovery Specialist** - Extract API interfaces from designs, UI-API mapping
- **🎨 Design Style Manager** - Centralized style system and brand consistency
- **📋 Documentation Auditor** - Compliance verification and documentation quality
- **🖼️ Wireframe Generator** - Create interactive wireframes and prototypes
- **🏛️ Enterprise Architect** - Complex system design and integration patterns
- **🔍 UX Analyzer** - User experience analysis and optimization
- **🚀 Prototype Developer** - **App requirements → prototypes + design spec transformation**

### 📦 Product Department (Stage 2)
- **👔 Product Lead** - MVP definition, user stories, PRD
- **📅 Project Planner** - Roadmap, sprints, estimates
- **💼 Business Analyst** - Market analysis, revenue model, GTM (JSA)

### 💻 Development Department (Stage 3)
- **👨‍💼 Principal Developer** - Implementation planning, code review, quality gate
- **🧪 QA Engineer** - Test planning, reviews, integration testing, sign-off
- **🗄️ Database Developer** - Schema, models, migrations
- **⚙️ Backend Developer** - APIs, services, business logic
- **🎨 Frontend Developer** - Components, pages, state management
- **🔗 Integration Engineer** - Connects components, E2E verification

### 🚀 Release & Deployment Department (Stage 4)
- **📝 Technical Writer** - Architecture docs, guides, README
- **🔐 Security Engineer** - Security validation, compliance, scanning
- **📦 Release Engineer** - Versioning, changelog, artifacts
- **🚀 DevOps Engineer** - IaC, CI/CD, deployment, monitoring

### 🌐 Go Live & Operate (Stage 5 - Optional)
- **🚀 SRE Deploy Engineer** - Quick deploy to managed platforms
- **🛡️ SRE Ops Engineer** - Monitoring, alerting, incidents, SLOs

### 🔧 Utility Agents
- **🐛 Bug Fixer** - Standalone utility for systematic error diagnosis and fixing

---

## Commands Reference (59 Total)

### Core Project Management (8)
| Command | Purpose |
|---------|---------|
| `/ts-new-project <name>` | Start a new project |
| `/ts-status` | Check current project status |
| `/ts-view [section]` | View project file sections |
| `/ts-brief` | Get executive summary |
| `/ts-ask <question>` | Ask Founder-Advisor a question |
| `/ts-exec-summary` | Full executive summary |
| `/ts-approve <gate>` | Approve at HITL gate |
| `/ts-review <stage>` | Request stage review |

### Stage 1: Architecture (2)
| Command | Agent | Purpose |
|---------|-------|---------|
| `/ts-assess` | Solution Architect | Assess project requirements and recommend architecture; Analyze existing/legacy codebases |
| `/ts-architect` | Solution Architect | Run architecture design phase |

### Stage 1.5: Design (8)
| Command | Agent | Purpose |
|---------|-------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vedanta/the-system](https://github.com/vedanta/the-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
