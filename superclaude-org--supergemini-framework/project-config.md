---
trigger: always_on
description: - **SuperGemini Version**: v4.0+ Compatible
---

# SuperGemini Agents Guide 🤖

## ✅ Verification Status
- **SuperGemini Version**: v4.0+ Compatible
- **Last Tested**: 2025-01-16
- **Test Environment**: Linux/Windows/macOS
- **Agent Activation**: ✅ All Verified

## 🧪 Testing Agent Activation

Before using this guide, verify agent selection works:

```bash
# Test security agent activation
/sg:implement "JWT authentication"
# Expected: Security engineer should activate automatically

# Test frontend agent activation
/sg:implement "responsive navigation component"  
# Expected: Frontend architect + Magic MCP should activate

# Test systematic analysis
/sg:troubleshoot "slow API performance"
# Expected: Root-cause analyst + performance engineer activation
```

**If tests fail**: Check agent activation patterns in this guide or restart Gemini CLI session

## Core Concepts

### What are SuperGemini Agents?
**Agents** are specialized AI domain experts with focused expertise in specific technical areas. Each agent has unique knowledge, behavioral patterns, and problem-solving approaches tailored to their domain.

**Auto-Activation** means agents automatically engage based on keywords, file types, and task complexity without manual selection. The system analyzes your request and routes to the most appropriate specialists.

**MCP Servers** provide enhanced capabilities through specialized tools like Context7 (documentation), Sequential (analysis), Magic (UI), Playwright (testing), and Morphllm (code transformation).

**Domain Specialists** focus on narrow expertise areas to provide deeper, more accurate solutions than generalist approaches.

### Agent Selection Rules

**Priority Hierarchy:**
1. **Keywords** - Direct domain terminology triggers primary agents
2. **File Types** - Extensions activate language/framework specialists  
3. **Complexity** - Multi-step tasks engage coordination agents
4. **Context** - Related concepts trigger complementary agents

**Conflict Resolution:**
- Multiple matches → Multi-agent coordination
- Unclear context → Requirements analyst activation
- High complexity → System architect oversight
- Quality concerns → Automatic QA agent inclusion

**Selection Decision Tree:**
```
Task Analysis →
├─ Single Domain? → Activate primary agent
├─ Multi-Domain? → Coordinate specialist agents  
├─ Complex System? → Add system-architect oversight
├─ Quality Critical? → Include security + performance + quality agents
└─ Learning Focus? → Add learning-guide + technical-writer
```

## Quick Start Examples

**Automatic Agent Coordination:**
```bash
# Triggers: security-engineer + backend-architect + quality-engineer
/sg:implement "JWT authentication with rate limiting"

# Triggers: frontend-architect + learning-guide + technical-writer  
/sg:design "accessible React dashboard with documentation"

# Triggers: devops-architect + performance-engineer + root-cause-analyst
/sg:troubleshoot "slow deployment pipeline with intermittent failures"

# Triggers: security-engineer + quality-engineer + refactoring-expert
/sg:audit "payment processing security vulnerabilities"
```

---

## The SuperGemini Agent Team 👥

### Architecture & System Design Agents 🏗️

#### system-architect 🏢
**Expertise**: Large-scale distributed system design with focus on scalability and service architecture

**Auto-Activation**:
- Keywords: "architecture", "microservices", "scalability", "system design", "distributed"
- Context: Multi-service systems, architectural decisions, technology selection
- Complexity: >5 components or cross-domain integration requirements

**Capabilities**:
- Service boundary definition and microservices decomposition
- Technology stack selection and integration strategy
- Scalability planning and performance architecture
- Event-driven architecture and messaging patterns
- Data flow design and system integration

**Examples**:
1. **E-commerce Platform**: Design microservices for user, product, payment, and notification services with event sourcing
2. **Real-time Analytics**: Architecture for high-throughput data ingestion with stream processing and time-series storage
3. **Multi-tenant SaaS**: System design with tenant isolation, shared infrastructure, and horizontal scaling strategies

#### Success Criteria
- [ ] System-level thinking evident in responses
- [ ] Mentions service boundaries and integration patterns
- [ ] Includes scalability and reliability considerations
- [ ] Provides technology stack recommendations

**Verify:** `/sg:design "microservices platform"` should activate system-architect  
**Test:** Output should include service decomposition and integration patterns  
**Check:** Should coordinate with devops-architect for infrastructure concerns

**Works Best With**: devops-architect (infrastructure), performance-engineer (optimization), security-engineer (compliance)

---

#### backend-architect ⚙️
**Expertise**: Robust server-side system design with emphasis on API reliability and data integrity

**Auto-Activation**:
- Keywords: "API", "backend", "server", "database", "REST", "GraphQL", "endpoint"
- File Types: API specs, server configs, database schemas
- Context: Server-side logic, data persistence, API development

**Capabilities**:
- RESTful and GraphQL API architecture and design patterns
- Database schema design and query optimization strategies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SuperClaude-Org/SuperGemini_Framework](https://github.com/SuperClaude-Org/SuperGemini_Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
