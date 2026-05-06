---
trigger: always_on
description: **AI Robo Advisor** is a Python-based intelligent financial assistant using **LangChain**, **LangGraph**, and multiple **LLM providers** to offer explainable investment insights and ETF portfolio analysis.
---

# 🤖 GitHub Copilot Instructions — AI Robo Advisor

## 🧭 Project Overview

**AI Robo Advisor** is a Python-based intelligent financial assistant using **LangChain**, **LangGraph**, and multiple **LLM providers** to offer explainable investment insights and ETF portfolio analysis.

**Goal:** Create a modular, multi-agent architecture for AI-powered financial advisory agents that combine LLM reasoning with structured market data.

---

## 🏗️ Architecture

### Multi-Agent Pipeline
```
User Input (Questionnaire)
    ↓
Investment Agent (goal_based.py) — Creates investment strategy
    ↓
Portfolio Agent (portfolios_agent.py) — Generates ETF portfolio
    ↓
Analyst Agents (Parallel Execution)
    ├─ Fees Agent (analyze_ter)
    ├─ Diversification Agent (analyze_diversification) 
    ├─ Alignment Agent (analyze_alignment)
    └─ Performance Agent (analyze_performance)
    ↓
Analysis Orchestrator (is_approved) — Aggregates results
    ↓
Final Report & Recommendations
```

### Key Principles
- **State-Driven**: All agents take and return `State` (TypedDict with `data` and `metadata`)
- **Function-Based**: Agents are functions, not classes
- **LLM-Powered**: Use structured outputs via Pydantic models
- **Parallel Execution**: Analyst agents run concurrently
- **Modular**: Easy to add new agents (see [AGENTS.md](../AGENTS.md))

---

## 📁 Project Structure

| Directory/File | Description |
|----------------|-------------|
| `src/main.py` | Application entry point |
| `src/nodes/investment_agents/` | Investment strategy creation |
| `src/nodes/portfolios_agent.py` | ETF portfolio generation |
| `src/nodes/analyst_agents/` | Portfolio analysis specialists |
| `src/graph/state.py` | State definition for LangGraph |
| `src/data/models.py` | Pydantic models |
| `src/llm/` | LLM provider abstraction |
| `src/tools/` | External API integrations (Polygon.io) |
| `src/utils/` | Helper utilities |
| `tests/` | Unit and integration tests |

---

## 🧩 Coding Conventions

**Language:** Python ≥ 3.10  
**Formatting:** Black, PEP8  
**Typing:** Required  
**Docstrings:** Google-style  
**Testing:** pytest  

### File Naming
| Type | Pattern | Example |
|------|---------|---------|
| Modules | `snake_case.py` | `portfolios_agent.py` |
| Tests | `test_<module>.py` | `test_analyst_workflow.py` |
| Agents | `analyze_{feature}()` | `analyze_ter()` |

---

## 🔧 Core Patterns

### Agent Pattern
```python
from graph.state import State
from data.models import AnalysisAgent

def analyze_feature(state: State) -> State:
    """Analyze specific aspect of portfolio."""
    llm = state["metadata"]["analyst_llm_agent"]
    portfolio = state["data"]["portfolio"]
    
    if "analysis" not in state["data"]:
        state["data"]["analysis"] = {}
    
    prompt = f"Analyze {portfolio.holdings} for specific criteria..."
    response = llm.with_structured_output(AnalysisAgent).invoke(prompt)
    
    if state["metadata"]["show_reasoning"]:
        print(response.reasoning)
    
    state["data"]["analysis"]["feature"] = response
    return state
```

### State Structure
```python
state = {
    "data": {
        "portfolio": Portfolio,
        "benchmark": tuple,
        "analysis": {
            "expense_ratio": AnalysisAgent,
            "diversification": AnalysisAgent,
            "alignment": AnalysisAgent,
            "performance": AnalysisAgent,
            "is_approved": bool,
            "summary": AnalysisResponse
        }
    },
    "metadata": {
        "show_reasoning": bool,
        "investment_llm_agent": LLM,
        "portfolio_llm_agent": LLM,
        "analyst_llm_agent": LLM
    }
}
```

### Pydantic Models
```python
from pydantic import BaseModel
from typing import Optional

class Status(BaseModel):
    key: str        # e.g., "is_cheaper"
    value: bool     # True if passes, False if fails

class AnalysisAgent(BaseModel):
    status: Status
    reasoning: str
    advices: Optional[list[str]]
```

---

## 🚀 Quick Setup

### Environment
```bash
git clone https://github.com/matvix90/ai-robo-advisor.git
cd ai-robo-advisor
python -m venv venv && source venv/bin/activate
pip install -e ".[dev]"
```

### Environment Variables
```bash
# .env file
OPENAI_API_KEY="sk-..."
POLYGON_API_KEY="your_key"
```

### Run & Test
```bash
run-advisor                     # Normal mode
run-advisor --show-reasoning    # Debug mode
pytest --cov=src               # Run tests with coverage
```

---

## 🧪 Testing Requirements

### Essential Commands
```bash
pytest -v                      # All tests
pytest tests/test_nodes.py -v  # Specific file
pytest --cov=src --cov-report=term-missing  # Coverage
black src tests                # Format code
```

### Test Pattern
```python
def test_analyze_feature_passes(sample_state):
    """Test that feature analysis passes with valid data."""
    mock_response = AnalysisAgent(
        status=Status(key="is_feature", value=True),
        reasoning="Test reasoning",
        advices=[]
    )
    
    mock_llm = Mock()
    mock_llm.with_structured_output.return_value.invoke.return_value = mock_response
    sample_state["metadata"]["analyst_llm_agent"] = mock_llm
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matvix90/ai-robo-advisor](https://github.com/matvix90/ai-robo-advisor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
