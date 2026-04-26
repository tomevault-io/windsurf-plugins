---
trigger: always_on
description: The FTE+AI Program Framework is a comprehensive 30-60-90 day program execution system designed to help R&D teams replace outsourcing vendors with AI-augmented internal teams. This is **NOT a documentation toolkit**—it is a hands-on program management framework with specialized AI agents that guide you through planning, execution, and successful cutover.
---

# FTE+AI Program Framework - AI Agent Guide

## Project Overview

The FTE+AI Program Framework is a comprehensive 30-60-90 day program execution system designed to help R&D teams replace outsourcing vendors with AI-augmented internal teams. This is **NOT a documentation toolkit**—it is a hands-on program management framework with specialized AI agents that guide you through planning, execution, and successful cutover.

**Primary Goal:** Enable R&D organizations to reduce vendor costs by 60-80% while increasing FTE productivity by 1.5-2.5x through strategic AI augmentation—delivered in 90 days.

**Mission:** Provide complete program orchestration from initial planning through vendor cutover, with AI agents that actively guide each phase of execution.

## Technology Stack & Architecture

**System Type:** AI-Powered Program Execution Framework  
**Platform:** GitHub-based with structured agent orchestration  
**Agent Framework:** Custom chatagent format with YAML frontmatter  
**Content Format:** Markdown templates, checklists, and tracking tools  
**Version Control:** Git-based with GitHub repository structure  
**Tools Framework:** Configurable tool access for program management  
**Execution Model:** 30-60-90 day phased program delivery  
**Current Version:** 4.1.0 (Production-Ready Agent Framework)

### Core Architecture Components

1. **Agent Layer** (`agents/`): 18 specialized AI agents orchestrating program phases
2. **Skill Layer** (`skills/`): 23 reusable expertise modules for program execution  
3. **Program Framework**: 30-60-90 day structured execution model
4. **Phase Gates**: Go/No-Go decision points at Days 30, 60, 90
5. **Tracking System**: Milestone, deliverable, risk, and stakeholder management
6. **Reference Layer** (root files): Program guides and execution templates

### Modern Agent Patterns (v4.1)

This framework implements cutting-edge agent architecture patterns based on 2024-2025 best practices:

| Pattern | Description | Used By |
|---------|-------------|---------|
| **Orchestrator-Workers** | Central coordinator delegates to specialists | Program-Manager |
| **Hub-and-Spoke** | Central hub synthesizes from multiple sources | Documaster |
| **Evaluator-Optimizer** | Generate-critique-improve loops | Tool-Evaluation-Specialist |
| **Pipeline** | Sequential processing through specialists | Implementation-Guide |
| **Specialist** | Deep expertise in narrow domain | ROI-Calculator, MLOps-Engineer |

### Agent Metadata Standard (v2.0)

All agents now include enhanced YAML frontmatter:

```yaml
---
description: 'Agent purpose description'
tools: ["Tool1", "Tool2"]
version: '2.0.0'
updated: '2025-12-31'
category: 'orchestration|documentation|financial|etc'
---
```

## 🛠️ Tools Configuration

Each agent has configured tool access appropriate for their specialized functions.

### Available Tools
- **File System**: `ReadFile`, `WriteFile`, `StrReplaceFile`, `Glob` - Document manipulation
- **System Operations**: `Shell`, `Grep` - Command execution and text processing  
- **Research**: `SearchWeb`, `FetchURL` - Internet research and data gathering
- **Parallel Processing**: `Task` - Complex task decomposition and execution

### Agent Tool Access Summary
| Agent | Primary Tools | Enterprise Focus |
|-------|---------------|------------------|
| Documaster | File system + Research | Documentation creation and research |
| ROI Calculator | File + Shell + Research | Financial modeling and market analysis |
| Implementation Guide | File system + Shell | Tutorial creation with code testing |
| Case Study Documenter | File + Research | Success story creation with data |
| Vendor Transition Manager | File system + Research | Project management and planning |
| Tool Evaluation Specialist | Research focused | Market analysis and comparisons |
| Security-Risk-Compliance Advisor | File + Research | Compliance research and frameworks |
| Change Management Coach | File + Research | Training materials and surveys |
| Performance Optimization Agent | File + Shell | Data analysis and system integration |
| Executive Strategy Advisor | File + Research | Market intelligence and strategy |
| API Integration Specialist | File + Shell + Research | Technical integration guides |
| Legal Contract Advisor | File + Research | Contract analysis and compliance |
| Local AI Infrastructure Architect | File + Shell | Hardware sizing and deployment |
| Open Source Model Evaluator | File + Research | Model selection and licensing |
| Data Sovereignty Advisor | File + Research | Compliance and data protection |
| Vendor Relationship Manager | File + Research | Contract negotiation and relations |
| MLOps Engineer | File + Shell + Grep | Production operations and monitoring |

### Security Considerations
- **Principle of Least Privilege**: Agents only have tools needed for their functions
- **Audit Logging**: All tool usage is logged for security monitoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitkox/fteplusai](https://github.com/mitkox/fteplusai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
