---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LieGraph is an AI-powered implementation of the social deduction game "Who Is Spy" built with LangGraph. It features autonomous AI agents that use LLM reasoning to find the spy among them.

- **Main Language**: Python 3.12+
- **Core Framework**: LangGraph for workflow orchestration
- **AI Integration**: LangChain with structured LLM outputs
- **Frontend**: React 19.2 with LangGraph SDK

## Development Commands

### Initial Setup
```bash
# Install Python dependencies (uses uv package manager)
uv sync

# Create .env file from template
cp .env.template .env
# Edit .env with your API keys (OpenAI, DeepSeek, or OpenRouter)

# Install frontend dependencies
cd ui-web/frontend && npm install
```

### Running the Application
```bash
# Terminal 1: Start LangGraph backend (from project root)
langgraph dev --config langgraph.json --port 8124 --allow-blocking

# Terminal 2: Start React frontend (from ui-web/frontend)
npm start

# Access UI at: http://localhost:3000
```

### Testing
```bash
# Run all tests
python -m pytest tests/ -v

# Run specific test file
python -m pytest tests/test_game_rules.py -v

# Run specific test
python -m pytest tests/test_game_rules.py::test_assign_roles -v
```

### Linting/Formatting
```bash
# Format Python code
black src/ tests/
```

## Architecture Overview

### LangGraph Workflow Flow
```
START → host_setup → host_stage_switch
                        ↓
                    speaking phase (sequential player_speech nodes)
                        ↓
                    voting phase (concurrent player_vote nodes)
                        ↓
                    check_votes_and_transition
                        ↓
                    host_result → (continue or END)
```

### Key Architectural Patterns

1. **State Management**: TypedDict-based GameState with private state separation
   - `GameState`: Shared public state (speech history, votes, game status)
   - `HostState`: Private host mindset (invariant after setup)
   - `PlayerState`: Private player mindsets (evolving beliefs about identities)

2. **Concurrent Voting**: Multiple players vote in parallel using LangGraph reducers
   - Reducer: `merge_votes` handles timestamp-based conflict resolution
   - Each vote node is independent but writes to shared state

3. **AI Strategy System** (`src/game/strategy/`):
   - `strategy_core.py`: Main LLM coordination
   - `builders/`: Context and prompt builders for speech/voting/inference
   - `llm_schemas.py`: Pydantic models for structured LLM outputs

4. **Agent Tools** (`src/game/agent_tools/`):
   - `speech_tools.py`: Structured reasoning for speech generation
   - `vote_tools.py`: Evidence-based voting decisions
   - Uses TrustCall for reliable structured output extraction

5 **Reducers**: State conflict resolution functions
   - `merge_private_states`: Combine incremental mindset updates
   - `merge_votes`: Handle concurrent vote submissions
   - Use `add` for append-only collections (speeches, votes)

6. **Conditional Routing**: `src/game/graph.py` uses dynamic edge routing
   - `host_stage_switch`: Routes between speaking and voting phases
   - Checks `current_speaker` and vote readiness
   - Returns edge names for graph transitions

7. **PyDict Structured Output**: Uses dict exports for serialization
   - GameState uses `PyDict` export methods for LangGraph checkpoints
   - Ensure all Pydantic models in state have proper serialization

8 **Private State Updates**: Player/hod nodes return private state deltas
   - Add "_" prefix: `{player_name: PlayerState}` → `{"_" + player_name: PlayerState}`
   - Host returns `f"_{HOST_NAME}": HostState`
   - Graph middleware merges private states using configured reducers

9. **Channel Configuration**: Define channels for each node in graph
   - Player nodes: `"_" + player_name` channels
   - Host node: `"_" + HOST_NAME` channel
   - Channels must match reducer keys

## Configuration

**LLM Configuration** (`.env`):
- Supports OpenAI, DeepSeek, and OpenRouter providers
- Set provider-specific API keys and model names
- Example models: `gpt-4o-mini`, `deepseek-chat`, `anthropic/claude-sonnet-4.5`

**Game Configuration** (`config.yaml`):
- `player_count`: Number of players (3-8)
- `vocabulary`: Word pairs for civilian/spy assignments
- `player_names`: Pool of available player names
- `metrics.enabled`: Toggle metrics collection on/off

## Testing Strategy

**Test Coverage** (50 tests across 6 modules):
- `test_game_rules.py`: Core game logic, role assignment, win conditions
- `test_state.py`: State management and reducer functions
- `test_host_nodes.py`: Host node behavior and phase transitions
- `test_player_nodes.py`: Player speech and voting nodes
- `test_llm_strategy.py`: AI strategy builders and prompt generation
- `agents/test_speech_tools.py`: LLM tool behavior and structured outputs

**Key Testing Patterns**:
- Use fixtures for common GameState configurations
- Mock LLM responses for deterministic AI behavior tests
- Test both sequential (speaking) and concurrent (voting) nodes
- Verify private state updates and mindset evolution

## Metrics and Quality Tracking


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leslieo2/LieGraph](https://github.com/leslieo2/LieGraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
