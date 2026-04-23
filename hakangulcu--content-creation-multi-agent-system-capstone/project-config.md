---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Content Creation Multi-Agent System built with LangGraph and local Ollama models. The system orchestrates 6 specialized agents through a sequential pipeline to generate high-quality, SEO-optimized content completely offline.

## Essential Commands

### Setup and Dependencies
```bash
# Install dependencies
pip install -r requirements.txt

# Download required NLTK data
python -c "import nltk; nltk.download('punkt')"

# Setup Ollama (if not installed)
curl -fsSL https://ollama.com/install.sh | sh
ollama serve
ollama pull llama3.1:8b
```

### Running the System
```bash
# Basic demo
python main.py

# Interactive demo with multiple scenarios
python demo.py

# Run specific components for testing
python -c "from agents.research_agent import ResearchAgent; print('Research agent imported successfully')"
```

### Testing
```bash
# Run all tests
pytest test_agents.py -v

# Run specific test categories
pytest test_agents.py::TestTools -v
pytest test_agents.py::TestAgents -v
pytest test_agents.py::TestWorkflow -v

# Run with coverage
pytest test_agents.py --cov=. --cov-report=html
```

### Code Quality
```bash
# Format code
python -m black .

# Lint code
python -m flake8

# Type checking (if mypy is installed)
mypy main.py agents/
```

## System Architecture

### Core Components
- **main.py** - Primary workflow orchestrator using LangGraph StateGraph
- **agents/** - 6 specialized agent modules (research, planning, writing, editing, SEO, QA)
- **demo.py** - Interactive demonstration system with 4 predefined scenarios
- **test_agents.py** - Comprehensive test suite with unit, integration, and performance tests

### Agent Pipeline Flow
1. **ResearchAgent** - Web search via DuckDuckGo, data gathering
2. **PlanningAgent** - Content structure, outline, keyword strategy  
3. **WriterAgent** - Content generation using Ollama LLM
4. **EditorAgent** - Quality improvement, readability optimization via NLTK
5. **SEOAgent** - Keyword analysis, SEO scoring
6. **QualityAssuranceAgent** - Final validation, file output

### State Management
The system uses a `ContentCreationState` TypedDict that flows through all agents:
```python
state = {
    "request": ContentRequest,
    "research_data": ResearchData, 
    "content_plan": ContentPlan,
    "draft": ContentDraft,
    "analysis": ContentAnalysis,
    "final_content": str,
    "feedback_history": List[str],
    "revision_count": int,
    "metadata": Dict[str, Any]
}
```

## Configuration

### Ollama Models
- Default: `llama3.1:8b` (8GB RAM required)
- Fast: `phi3:mini` (4GB RAM required)  
- High-quality: `llama3.1:70b` (64GB RAM required)

### Environment Variables (.env)
```env
OLLAMA_MODEL=llama3.1:8b
OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_TEMPERATURE=0.7
OLLAMA_NUM_PREDICT=4096
```

## Development Patterns

### Adding New Agents
1. Create new agent file in `agents/` directory
2. Inherit common patterns from existing agents
3. Add to `agents/__init__.py` imports
4. Register in workflow graph in `main.py:_build_workflow()`
5. Add corresponding tests in `test_agents.py`

### Tool Development
Tools use `@tool` decorator from langchain_core:
```python
@tool
def custom_tool(input: str) -> dict:
    """Tool description for LLM"""
    # Implementation
    return {"result": "processed"}
```

### Error Handling
- All agents include try/catch blocks with fallback responses
- Web search tools handle network failures gracefully  
- File operations create directories automatically
- LLM calls have timeout and retry logic

### Testing Strategy
- Mock Ollama responses for consistent unit testing
- Use `pytest-asyncio` for async agent testing
- Performance tests measure end-to-end pipeline timing
- Edge case tests cover invalid inputs and resource constraints

## Common Issues

### Ollama Connection
If "Connection refused" errors occur:
```bash
# Check Ollama status
ollama list
ollama serve

# Test connection
curl http://localhost:11434/api/tags
```

### Memory Issues
For "Out of memory" errors, switch to smaller model:
```bash
ollama pull phi3:mini
# Update .env: OLLAMA_MODEL=phi3:mini
```

### NLTK Data Missing
If content analysis fails:
```bash
python -c "import nltk; nltk.download('punkt')"
```

## Detailed File Structure

### Root Directory Files

#### Documentation Files
- **CLAUDE.md** - AI assistant guidance and project instructions (this file)
- **README.md** - Comprehensive project overview and setup instructions
- **API_DOCUMENTATION.md** - API reference and integration details
- **AGENT_DOCUMENTATION.md** - Detailed agent specifications and workflows
- **PROJECT_STRUCTURE.md** - Project architecture and component breakdown
- **SYSTEM_ARCHITECTURE.md** - System design and technical architecture
- **SUBMISSION_CHECKLIST.md** - Project completion and submission requirements
- **OLLAMA_SETUP_GUIDE.md** - Step-by-step Ollama installation and configuration
- **LICENSE** - MIT license for the project

#### Core System Files
- **main.py** - Primary workflow orchestrator using LangGraph StateGraph
- **demo.py** - Interactive demonstration system with 4 predefined scenarios

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hakangulcu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
