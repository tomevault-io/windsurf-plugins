---
trigger: always_on
description: InfraX is an intelligent Infrastructure as Code (IaC) orchestration platform powered by "Brahma" - a multi-agent AI system that automates cloud infrastructure management using GPT-4.
---

# InfraX - Brahma: Intelligent IaC Orchestration Platform

## Project Overview
InfraX is an intelligent Infrastructure as Code (IaC) orchestration platform powered by "Brahma" - a multi-agent AI system that automates cloud infrastructure management using GPT-4.

**Current Version:** 1.0 (Production-Ready for Code Generation)

---

## Brahma Architecture

### Core Components (All Operational ✅)

**1. Intelligent Planner Tool**
- Location: `backend/brahma/tools/intelligent_planner.py`
- Purpose: Analyzes natural language prompts and plans complete infrastructure
- Features:
  - Location-aware cloud provider selection (AWS/Azure/GCP)
  - Automatic region optimization for lowest latency
  - Complete architecture design
  - Service mapping for all components
  - Multi-cloud region knowledge base

**2. Service Selection Agent**
- Location: `backend/brahma/agents/service_selection.py`
- Purpose: Recommends optimal cloud services
- Features:
  - 200+ cloud services across AWS, Azure, GCP
  - Workload-specific recommendations
  - Service comparison and alternatives
  - Best practices application
  - Provider comparison mode

**3. Cost Optimization Agent**
- Location: `backend/brahma/agents/cost_optimization.py`
- Purpose: Analyzes costs and identifies savings (40-60% target)
- Features:
  - AI-powered cost analysis
  - Right-sizing recommendations
  - Reserved instance strategies
  - Storage tiering optimization
  - 12-month cost forecasting
  - Resource scheduling strategies

**4. IaC Generation Agent**
- Location: `backend/brahma/agents/iac_generation.py`
- Purpose: Generates production-ready Terraform code
- Features:
  - Terraform code only (fixed)
  - Security best practices built-in
  - Multi-AZ deployment configurations
  - Auto-scaling setup
  - Monitoring and logging included
  - Supports AWS, Azure, GCP

**5. Diagram Generator Tool**
- Location: `backend/brahma/tools/diagram_generator.py`
- Purpose: Creates visual architecture diagrams
- Features:
  - Mermaid diagram generation (React-compatible)
  - Dark glassmorphism theme matching frontend
  - Network topology visualization
  - Service relationship mapping
  - Data flow diagrams
  - AI-generated detailed service descriptions
  - Interactive hover tooltips with requirements
  - Interactive HTML preview
  - GitHub/GitLab compatible

**6. Cloud Pricing Tool** (NEW ✨)
- Location: `backend/brahma/tools/cloud_pricing.py`
- Purpose: Provides accurate cloud service pricing
- Features:
  - AI-powered pricing estimates (OpenAI GPT-4)
  - Real-time pricing knowledge (up to Jan 2025)
  - AWS, Azure, GCP pricing support
  - Regional pricing variations
  - Instance type recommendations
  - Multi-cloud price comparison
  - Cost breakdown by component (compute, storage, network)
  - Optimization tips included

**7. Unified Orchestrator**
- Location: `backend/brahma/core/orchestrator.py`
- Purpose: Coordinates all agents and tools
- Features:
  - Intelligent workflow mode (auto-planning)
  - Standard workflow mode (manual control)
  - 5-step automated process
  - Workflow history tracking
  - PostgreSQL persistence
  - Complete result packaging

---

## Project Structure

```
InfraX/
├── backend/
│   ├── brahma/                    # Brahma AI System
│   │   ├── agents/
│   │   │   ├── __init__.py
│   │   │   ├── service_selection.py
│   │   │   ├── cost_optimization.py
│   │   │   └── iac_generation.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   └── orchestrator.py
│   │   ├── tools/
│   │   │   ├── __init__.py
│   │   │   ├── intelligent_planner.py
│   │   │   ├── diagram_generator.py
│   │   │   └── cloud_pricing.py
│   │   ├── utils/
│   │   │   └── __init__.py
│   │   └── __init__.py
│   ├── api/
│   │   └── main.py                # FastAPI server + CORS
│   ├── database.py                # PostgreSQL models (SQLAlchemy)
│   ├── templates/
│   │   └── terraform/
│   │       └── aws_base.tf
│   ├── requirements.txt
│   ├── cli.py                     # Demo CLI (not active)
│   └── .env.example
├── frontend/                       # React + TypeScript (NEW ✨)
│   ├── src/
│   │   ├── components/
│   │   │   ├── MermaidDiagram.tsx  # Dark themed diagrams
│   │   │   ├── Sidebar.tsx
│   │   │   └── DashboardLayout.tsx
│   │   ├── pages/
│   │   │   ├── Dashboard.tsx
│   │   │   ├── CreateWorkflow.tsx
│   │   │   ├── Workflows.tsx
│   │   │   └── WorkflowDetails.tsx
│   │   ├── api/
│   │   │   └── brahma.ts          # API client
│   │   ├── types/
│   │   │   └── workflow.ts        # TypeScript definitions
│   │   └── App.tsx
│   ├── package.json
│   └── tailwind.config.js         # Dark glassmorphism theme
├── data/
│   ├── generated_code/            # Terraform files output
│   └── diagrams/                  # Mermaid diagrams + HTML previews
├── PROJECT_VISION.txt             # Complete project vision document
└── CLAUDE.md                      # This file (to be gitignored)
```

---

## Tech Stack

### Backend
- **Runtime:** Python 3.8+
- **Framework:** FastAPI (REST API) + CORS
- **AI:** OpenAI GPT-4 (all agents and tools)
- **IaC Output:** Terraform only
- **Visualization:** Mermaid.js (dark themed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sushilpandeyy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
