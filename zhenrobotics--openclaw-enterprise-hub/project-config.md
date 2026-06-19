---
trigger: always_on
description: Enterprise orchestration layer for cross-system permission coordination, workflow automation, and data consistency management
---


# Enterprise Agent OS Skill

**The Orchestration Layer for Enterprise Software**

Control cross-system workflows. Coordinate permissions across 20+ enterprise systems. Own the enterprise budget.

---

## Strategic Positioning

### The Power Transfer

Enterprise software power is shifting from **application layer** to **orchestration layer**.

**Past 20 Years**: Salesforce, SAP, Workday ruled independently
**Next 10 Years**: Orchestration platforms control workflows across all systems

**Enterprise Agent OS positions you at this critical inflection point.**

---

## Core Capabilities

### 1. Permission Topology Orchestration ⭐ MVP Focus

**The Problem Nobody Else Solves:**
```
Employee has Salesforce access to "Customer A"
BUT no SAP access to "Customer A" financial data

Traditional solution: Manual IT ticket → 3-day delay
Our solution: Real-time cross-system permission coordination → < 50ms
```

**What It Does:**
- Queries permissions across all connected systems simultaneously
- Calculates minimum permission intersection
- Detects and resolves permission conflicts automatically
- Provides complete audit trail for compliance

**Business Impact:**
- 70% reduction in IT permission tickets
- Zero manual escalation for standard requests
- Complete compliance audit trail

### 2. Data Consistency Management

**Enterprise Event Sourcing** - Single source of truth for all system changes

**What It Does:**
- Every system change flows through central event log
- Automatic conflict detection when systems diverge
- Conflict resolution rules (last-write-wins, custom merge, manual)
- Complete replay capability for debugging

**Business Impact:**
- 99.9% data consistency across systems
- Zero manual data reconciliation
- 7-year audit trail retention

### 3. Fault Isolation & Graceful Degradation

**The Problem:**
```
Integration hub fails → 20 systems lose coordination → Operations paralyzed
```

**Our Solution:**
- Each system continues operating independently during hub downtime
- Automatic queuing of pending operations
- Intelligent reconciliation on recovery
- Zero operational interruption

**Business Impact:**
- 99.9% system uptime
- Zero revenue loss from integration failures

---

## When to Use This Skill

### AUTO-TRIGGER when user's message contains:

**Permission Management Keywords:**
- "check permissions", "permission conflict", "cross-system access"
- "who has access to", "grant access across", "permission audit"
- "compliance report", "access control"

**Workflow Orchestration Keywords:**
- "automate workflow", "cross-system workflow", "enterprise automation"
- "integrate Salesforce and SAP", "sync data across systems"
- "customer onboarding process", "employee offboarding"

**System Integration Keywords:**
- "connect enterprise systems", "integrate CRM and ERP"
- "data consistency", "system synchronization"
- "single source of truth"

**Enterprise Context:**
- User mentions multiple enterprise systems (Salesforce + SAP + Workday)
- Requests involving 100+ employees or enterprise-scale operations
- Questions about compliance, audit trails, or security

### TRIGGER EXAMPLES:

1. "Check if Alice has permission to view Customer A data across all systems"
2. "Create a workflow to onboard new customers across Salesforce, SAP, and Jira"
3. "Audit who has access to financial records in the last 30 days"
4. "Sync customer data between CRM and ERP automatically"
5. "What happens if the integration hub fails during a critical operation?"

### DO NOT USE when:

- Simple single-system tasks (use native tools)
- Personal productivity workflows (use Zapier/n8n)
- Video generation, media processing (different domain)

---

## Installation & Setup

### Prerequisites

- Node.js >= 18.0.0
- PostgreSQL >= 14 (for event store)
- Redis >= 6 (for caching)
- Docker (optional, for containerized deployment)

### Step 1: Install the Skill

```bash
clawhub install enterprise-agent-os
```

### Step 2: Clone & Setup

```bash
# Clone project
git clone https://github.com/YourOrg/openclaw-enterprise-hub.git ~/enterprise-agent-os
cd ~/enterprise-agent-os

# Install dependencies
npm install

# Setup environment
cp .env.example .env
nano .env  # Configure database, Redis, API keys
```

### Step 3: Database Setup

```bash
# Create PostgreSQL database
createdb enterprise_agent_os

# Run migrations
npm run db:migrate

# Seed initial data (optional)
npm run db:seed
```

### Step 4: Start Services

```bash
# Development mode
npm run dev

# Production mode
npm run build
npm run start

# With Docker
docker-compose up -d
```

### Step 5: Verify Installation

```bash
# Check system health
curl http://localhost:3000/health

# Run integration tests
npm run test:integration

# Check API documentation
open http://localhost:3000/api-docs
```

---

## Configuration

### Environment Variables

```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/enterprise_agent_os
REDIS_URL=redis://localhost:6379

# Permission Engine
OPA_ENDPOINT=http://localhost:8181
PERMISSION_CACHE_TTL=300  # 5 minutes

# Connected Systems (add your enterprise systems)
SALESFORCE_CLIENT_ID=your_client_id
SALESFORCE_CLIENT_SECRET=your_secret

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhenRobotics/openclaw-enterprise-hub](https://github.com/ZhenRobotics/openclaw-enterprise-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
