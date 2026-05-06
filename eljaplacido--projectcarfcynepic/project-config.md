---
trigger: always_on
description: > Purpose: Operational context for AI coding assistants to safely modify the CARF codebase.
---

# AGENTS.md - AI Coding Context for CARF

> Purpose: Operational context for AI coding assistants to safely modify the CARF codebase.

---

## Project Overview

CARF (Complex-Adaptive Reasoning Fabric) is a neuro-symbolic-causal agentic system.

Core Architecture: 6-layer cognitive stack
1. Router (Layer 1): Cynefin classification → routes to appropriate solver
2. Cognitive Mesh (Layer 2): LangGraph agents (Deterministic, Causal, Bayesian, Circuit Breaker)
3. Causal World Model (Layer 3): SCMs, counterfactual engine, neurosymbolic reasoning, H-Neuron sentinel
4. Reasoning Services (Layer 4): Neo4j (causal graphs), Experience Buffer (semantic memory), Agent Memory (persistent), RAG (3-layer retrieval)
5. Guardian (Layer 5): Policy enforcement (YAML + CSL-Core + OPA), HumanLayer approval gates
6. Auth & Cloud (Layer 6): Firebase JWT, Cloud SQL, per-user history

---

## Critical Rules

### DO NOT TOUCH (Immutable Core)
- `src/core/state.py` - EpistemicState schema is the contract for all agents
- `config/policies.yaml` - Safety policies require human review
- Any file in `.github/workflows/` - CI/CD changes need human approval

### ALWAYS DO
- Update `CURRENT_STATUS.md` before starting any feature work
- Run `pytest tests/` before committing
- Include Pydantic schemas for all new tools
- Wrap external API calls in tenacity retry decorators
- Log all state transitions for audit trail

### EXPLAINABILITY REQUIREMENTS
- Every analytical result MUST link to its data source
- Confidence scores MUST be decomposable (show what contributes)
- All panels MUST answer: "Why this?" + "How confident?" + "Based on what?"
- Drill-down capability MUST be available for all insights

---

## Testing Commands

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src --cov-report=term-missing

# Run only unit tests
pytest tests/unit/ -v

# Type checking
mypy src/ --strict

# Linting
ruff check src/ tests/
ruff format src/ tests/
```

---

## Environment Variables

Required:
```
LLM_PROVIDER=deepseek           # or "openai"
DEEPSEEK_API_KEY=               # DeepSeek API key (or OPENAI_API_KEY)
```

Optional:
```
OPENAI_API_KEY=                 # OpenAI fallback
HUMANLAYER_API_KEY=             # Human-in-the-loop
LANGSMITH_API_KEY=              # Tracing
CARF_TEST_MODE=1                # Offline LLM stubs for tests
CARF_API_URL=http://localhost:8000  # React Cockpit -> API target
CARF_DATA_DIR=./var             # Dataset registry storage (optional)
CARF_PROFILE=research           # Deployment profile: research | staging | production
CARF_API_KEY=                   # API key for staging/production auth
CARF_CORS_ORIGINS=              # Comma-separated CORS origins (overrides profile default)
CARF_MEMORY_DIR=data/memory     # Persistent agent memory storage
CARF_EMBEDDINGS_DIR=data/embeddings  # Numpy embedding cache
```

Phase 3/4 (optional):
```
NEO4J_URI=                      # bolt://localhost:7687
NEO4J_USERNAME=
NEO4J_PASSWORD=
NEO4J_DATABASE=neo4j

KAFKA_ENABLED=false
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
KAFKA_TOPIC=carf_decisions
KAFKA_CLIENT_ID=carf

OPA_ENABLED=false
OPA_URL=http://localhost:8181
OPA_POLICY_PATH=/v1/data/carf/guardian/allow
OPA_TIMEOUT_SECONDS=5
```

---

## Code Style Standards

### Pydantic Models
All data structures must use Pydantic `BaseModel`:
```python
from pydantic import BaseModel, Field

class ToolInput(BaseModel):
    """Input schema for my_tool."""
    query: str = Field(..., description="The search query")
    limit: int = Field(default=10, ge=1, le=100)
```

### Tenacity Retry Pattern
External calls must use retry decorators:
```python
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
async def call_external_api():
    ...
```

### LangGraph Nodes
All graph nodes must accept and return `EpistemicState`:
```python
def my_node(state: EpistemicState) -> EpistemicState:
    state.add_reasoning_step(
        node_name="my_node",
        action="Performed analysis",
        input_summary="...",
        output_summary="...",
    )
    return state
```

---

## Directory Structure (MECE)

```
projectcarf/
  carf-cockpit/           # React Platform Cockpit (Vite + TypeScript + Tailwind)
    src/
      components/carf/    # Core UI components (44 implemented)
        BayesianPanel.tsx, CausalAnalysisCard.tsx, CausalDAG.tsx,
        CynefinRouter.tsx, DashboardHeader.tsx, DashboardLayout.tsx,
        ExecutionTrace.tsx, GuardianPanel.tsx, QueryInput.tsx,
        ResponsePanel.tsx, SimulationArena.tsx, PolicyEditorModal.tsx,
        EscalationModal.tsx, FloatingChatTab.tsx, OnboardingOverlay.tsx,
        DataOnboardingWizard.tsx, ConversationalResponse.tsx,
        WalkthroughManager.tsx, MethodologyModal.tsx,
        ExecutiveSummaryPanel.tsx, AgentsInvolvedPanel.tsx,
        FeedbackPanel.tsx, DomainVisualization.tsx, ...
      __tests__/          # Frontend test suite (5 test files)
      services/           # API client layer
      hooks/              # Custom React hooks
      types/              # TypeScript type definitions
  src/
    core/               # Base classes, state schemas, deployment profiles (NO external deps)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eljaplacido/projectcarfcynepic](https://github.com/eljaplacido/projectcarfcynepic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
