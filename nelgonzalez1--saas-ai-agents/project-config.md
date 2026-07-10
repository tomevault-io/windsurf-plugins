---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a comprehensive collection of specialized Claude Code agents designed for **SaaS (Software as a Service) development**. These are expert-level AI agents that cover the complete SaaS development lifecycle, from initial planning through scaled production deployment.

**Purpose**: Accelerate SaaS development by providing expert guidance across all disciplines needed to build successful SaaS applications.

## Repository Structure

```
saas-development-agents/
├── README.md                    # Main project documentation
├── CLAUDE.md                    # This file - Claude Code guidance  
├── docs/                        # Comprehensive documentation
│   ├── getting-started.md       # How to create and use agents
│   ├── agent-overview.md        # Complete SaaS development lifecycle
│   └── contributing.md          # Future expansion guidelines
├── agents/                      # Organized agent templates
│   ├── core/                    # Essential SaaS development agents
│   │   ├── backend-engineer.json     # APIs, databases, server logic
│   │   ├── frontend-engineer.json    # React/Angular, UI components
│   │   ├── system-architect.json     # Technical architecture, tech stack
│   │   └── devops-engineer.json      # CI/CD, cloud infrastructure
│   ├── quality/                 # Quality & Security agents
│   │   ├── qa-testing.json           # Testing strategies, automation
│   │   └── security-analyst.json     # Security audits, vulnerabilities
│   ├── product/                 # Product & Design agents
│   │   ├── product-manager.json      # Requirements, MVPs, roadmaps
│   │   └── ux-ui-designer.json       # Design systems, user experience
│   └── specialized/             # Future specialized agents
│       └── .gitkeep             # Placeholder for expansion
├── examples/                    # Example SaaS projects (future)
└── scripts/                     # Utility scripts (future)
```

## Agent Template Format

Each agent template is a JSON file with the following structure:
- `name`: Agent identifier
- `description`: Use case and specialization
- `color`: Visual identifier
- `model`: Claude model to use
- `instructions`: Detailed behavioral instructions and expertise

## SaaS Development Focus

This agent collection is specifically designed for **SaaS application development** with the following considerations:

### SaaS-Specific Features
- **Multi-tenant architecture** patterns and implementations
- **Subscription billing** integration and management
- **Scalability planning** for rapid user growth
- **Security compliance** (SOC2, GDPR, HIPAA considerations)
- **Performance optimization** for concurrent users
- **API-first architecture** for integrations and mobile apps

### Technology Stack Coverage
Our agents support multiple technology stacks commonly used in SaaS:

#### Backend Technologies
- **Node.js** (Express, Fastify, NestJS) with PostgreSQL/MySQL
- **ASP.NET Core** with SQL Server/PostgreSQL  
- **Python** (Django, FastAPI) with PostgreSQL/MongoDB
- **Java** (Spring Boot) with PostgreSQL/MySQL

#### Frontend Technologies
- **React** with TypeScript (Redux, Context API, React Query)
- **Angular** with TypeScript (NgRx, RxJS)
- **Vue.js** (future support planned)

#### Cloud Platforms
- **AWS** (comprehensive examples and patterns)
- **Microsoft Azure** (enterprise-focused deployments)
- **Google Cloud Platform** (data-intensive applications)

## Working with SaaS Agent Templates

### Analyzing Agent Templates
When examining these files, focus on:
- **SaaS-specific challenges** addressed by each agent
- **Multi-technology examples** showing different implementation approaches
- **Scalability patterns** designed for growth
- **Security best practices** for SaaS applications
- **Integration patterns** with other agents in the collection

### Modifying Agent Templates  
When editing agent templates for SaaS projects:
1. **Preserve SaaS Focus**: Maintain emphasis on multi-tenancy, scalability, security
2. **Multi-Stack Support**: Include examples for different technology combinations
3. **Real-World Patterns**: Use patterns proven in production SaaS applications
4. **Integration Awareness**: Consider how agents work together in SaaS development
5. **Business Metrics**: Include relevant SaaS business considerations

### Creating New SaaS Agent Templates
Follow the established SaaS-focused pattern:
1. **Define SaaS Role**: Clear boundaries within SaaS development lifecycle
2. **SaaS Philosophy**: Include principles specific to SaaS challenges
3. **Multi-Technology Support**: Provide examples across major tech stacks
4. **Scalability Focus**: Address growth and scaling considerations
5. **Security Emphasis**: Include security patterns for SaaS applications
6. **Business Alignment**: Connect technical decisions to business outcomes

## SaaS Agent Specializations

### Core Development Agents (Essential for Every SaaS)
- **Backend Engineer**: Multi-tenant APIs, database design, authentication, subscription logic, performance optimization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nelgonzalez1/saas-ai-agents](https://github.com/nelgonzalez1/saas-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
