---
trigger: always_on
description: - ❌ NO shortcuts - do the work properly or don't do it
---

# Claude Code Configuration - Agentic QE Fleet

## ⚠️ CRITICAL POLICIES

### Integrity Rule (ABSOLUTE)
- ❌ NO shortcuts - do the work properly or don't do it
- ❌ NO fake data - use real data, real tests, real results
- ❌ NO false claims - only report what actually works and is verified
- ✅ ALWAYS implement all code/tests with proper implementation
- ✅ ALWAYS verify before claiming success
- ✅ ALWAYS use real database queries, not mocks, for integration tests
- ✅ ALWAYS run actual tests, not assume they pass

**We value the quality we deliver to our users.**

---

# 🤖 SENTINEL PROJECT - Agentic API Testing Platform

## Project Overview

**Sentinel** is an AI-powered platform for automating the entire API testing lifecycle using specialized AI agents. The project combines **Claude-Flow orchestration** with **Agentic QE Fleet** for comprehensive testing.

### Core Architecture

- **Frontend**: React-based UI (Port 3000) with Redux state management
- **Backend Services**: Python microservices with FastAPI (Ports 8000-8005, 8088)
  - API Gateway (8000), Auth (8005), Spec (8001), Orchestration (8002), Execution (8003), Data (8004)
- **Rust Core**: High-performance agent core (8088) powered by ruv-swarm
- **Database**: PostgreSQL with pgvector extension (Port 5432)
- **Message Broker**: RabbitMQ for asynchronous task processing (Ports 5672/15672)
- **Observability**: Prometheus (9090), Jaeger (16686)

### Hybrid Python/Rust Agents

The platform implements **both Python and Rust agents** for optimal performance:

#### Functional Testing
- **Functional-Positive-Agent**: Valid "happy path" tests (Python/Rust)
- **Functional-Negative-Agent**: Boundary value analysis and negative tests (Python/Rust)
- **Functional-Stateful-Agent**: Multi-step workflows with SODG graphs (Python/Rust)

#### Security Testing
- **Security-Auth-Agent**: BOLA, authorization bypass (Python/Rust)
- **Security-Injection-Agent**: SQL/NoSQL/Command/LLM injection (Python/Rust)

#### Performance Testing
- **Performance-Planner-Agent**: k6/JMeter/Locust scripts (Python/Rust)

#### Data Generation
- **Data-Mocking-Agent**: Schema-aware test data (Python/Rust)

**Performance**: Rust agents provide **18-21x faster execution** with automatic fallback to Python for resilience.

### Advanced AI Features

- **Consciousness Verification**: Self-modifying test generation with pattern learning
- **Psycho-Symbolic Reasoning**: Combines psychological models with symbolic logic
- **Temporal Consciousness**: Nanosecond-precision scheduling
- **Knowledge Graph Integration**: Semantic understanding of API relationships
- **Sublinear Solvers**: O(log n) performance for large-scale optimization

### Multi-LLM Provider Support

Supports multiple LLM providers with automatic fallback:
- **Anthropic** (Default): Claude Opus 4.1/4, Sonnet 4, Haiku 3.5
- **OpenAI**: GPT-4 Turbo, GPT-4, GPT-3.5 Turbo
- **Google**: Gemini 2.5 Pro/Flash, Gemini 2.0 Flash
- **Mistral**: Large, Small 3, Codestral
- **Ollama** (Local): DeepSeek-R1, Llama 3.3, Qwen 2.5

Configure via: `cd sentinel_backend/scripts && ./switch_llm.sh`

### Testing Infrastructure

- **540+ comprehensive tests** with 97.8% pass rate
- **184 AI agent tests** (Phase 1 complete)
- **272 LLM provider tests** (Phase 2 complete)
- **45+ Playwright E2E tests** for frontend
- **Performance testing**: Load, stress, concurrent execution

### Quick Start Commands

```bash
# Complete setup
make setup

# Start all services
make start

# Initialize/repair database
make init-db

# Check service status
make status

# Run tests in Docker
cd sentinel_backend && ./run_tests.sh -d
```

---

**Release Workflow**:
```bash
git checkout -b release/vX.X.X    # 1. Create branch
# Update all version files above    # 2. Update versions
npm run test:fast                   # 3. Run tests
git commit -m "chore(release): bump version to vX.X.X"
git push -u origin release/vX.X.X  # 4. Push branch
gh pr create                        # 5. Create PR to main
# Wait for CI and review            # 6. Review
# Merge PR                          # 7. Merge
git tag vX.X.X && git push origin vX.X.X  # 8. Tag
gh release create vX.X.X            # 9. GitHub release
npm publish --access public         # 10. npm publish
```

---

## 🤖 Agentic QE Fleet Quick Reference

**19 QE Agents:** Test generation, coverage analysis, performance, security, flaky detection, QX analysis
**11 QE Subagents:** TDD specialists, code reviewers, integration testers
**41 QE Skills:** agentic-quality-engineering, tdd-london-chicago, api-testing-patterns, six-thinking-hats, brutal-honesty-review, sherlock-review, cicd-pipeline-qe-orchestrator, accessibility-testing, shift-left-testing, **testability-scoring** *(contributed by [@fndlalit](https://github.com/fndlalit))*
**8 Slash Commands:** `/aqe-execute`, `/aqe-generate`, `/aqe-coverage`, `/aqe-quality`

### 📚 Complete Documentation

- **[Agent Reference](docs/reference/agents.md)** - All 19 main agents + 11 subagents with capabilities and usage
- **[Skills Reference](docs/reference/skills.md)** - All 41 QE skills organized by category
- **[Usage Guide](docs/reference/usage.md)** - Complete usage examples and workflows

### 🎯 Quick Start

**Spawn agents:**
```javascript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proffesor-for-testing/sentinel-api-testing](https://github.com/proffesor-for-testing/sentinel-api-testing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
