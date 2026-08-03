---
trigger: always_on
description: EurekaClaw has five specialized agents coordinated by the `MetaOrchestrator`. Each agent runs a tool-use loop with periodic context compression.
---

# Agents

EurekaClaw has five specialized agents coordinated by the `MetaOrchestrator`. Each agent runs a tool-use loop with periodic context compression.

## BaseAgent

All agents inherit from `eurekaclaw/agents/base.py`:

**Key Methods:**

| Method | Description |
|---|---|
| `execute(task: Task) -> AgentResult` | Abstract. Run the agent on a task |
| `get_tool_names() -> list[str]` | Abstract. Return allowed tool names for this agent |
| `build_system_prompt(task: Task) -> str` | Combine role prompt + injected skills |
| `run_agent_loop(task, initial_user_message, max_turns, max_tokens)` | Tool-use loop with context compression |
| `_compress_history() -> str` | Summarize conversation with fast model every N turns |
| `_call_model(system, messages, tools, max_tokens)` | LLM call with exponential-backoff retry |

**Context Compression:** Every `CONTEXT_COMPRESS_AFTER_TURNS` turns (default 6), history longer than 12 messages is compressed to a single summary using the fast model. This bounds input token growth across long runs.

---

## SurveyAgent

**Role:** `SURVEY`
**File:** `eurekaclaw/agents/survey/agent.py`
**Max Turns:** `SURVEY_MAX_TURNS` (default 8)

**Purpose:** Search the literature and populate the KnowledgeBus with papers, open problems, and key mathematical objects.

**Tools:**
- `arxiv_search` — Find relevant papers on arXiv
- `semantic_scholar_search` — Search Semantic Scholar for citation counts and metadata
- `web_search` — Supplement with general web search
- `citation_manager` — Format and store references

**Inputs (from KnowledgeBus):**
- `ResearchBrief.domain`
- `ResearchBrief.query`
- `ResearchBrief.conjecture`

**Outputs:**
- Appends papers to `Bibliography` on the bus
- Writes to `ResearchBrief`:
  - `open_problems` — list of identified open problems
  - `key_mathematical_objects` — core concepts and structures

**Output JSON keys:** `papers`, `open_problems`, `key_mathematical_objects`, `research_frontier`, `insights`

---

## IdeationAgent

**Role:** `IDEATION`
**File:** `eurekaclaw/agents/ideation/agent.py`
**Max Turns:** 3

**Purpose:** Generate 5 novel research hypotheses from survey findings. Each direction is scored on `novelty_score`, `feasibility_score`, and `impact_score` (mapped internally to the `ResearchDirection` fields `novelty_score`, `soundness_score`, `transformative_score`).

Direction *selection* does **not** happen inside IdeationAgent. After IdeationAgent writes `ResearchBrief.directions`, the orchestrator's `direction_selection_gate` task invokes `DivergentConvergentPlanner.converge()` to pick the highest-scoring direction and set `ResearchBrief.selected_direction`.

**Inputs (from KnowledgeBus):**
- Survey findings (`ResearchBrief`)
- `Bibliography`

**Outputs:**
- `ResearchBrief.directions` — 5 `ResearchDirection` objects with composite scores

---

## TheoryAgent

**Role:** `THEORY`
**File:** `eurekaclaw/agents/theory/agent.py`
**Max Iterations:** `THEORY_MAX_ITERATIONS` (default 10)
**Inner Stage Max Turns:** `THEORY_STAGE_MAX_TURNS` (default 6)

**Purpose:** Prove the selected research direction via a 7-stage bottom-up proof pipeline.

**Tools:**
- `arxiv_search` — Look up lemmas and techniques from papers
- `wolfram_alpha` — Symbolic computation and bound verification
- `lean4_verify` — Formal proof verification in Lean4
- `execute_python` — Numerical checks and sanity tests

**Inputs (from KnowledgeBus):**
- `ResearchBrief.selected_direction`

**Outputs:**
- `TheoryState` with `status` = `proved` / `refuted` / `abandoned`

**7-Stage Inner Loop** (`inner_loop_yaml.py`):

| Stage | Class | Input | Output |
|---|---|---|---|
| 1 | `PaperReader` | `Bibliography` | `known_results[]` |
| 2 | `GapAnalyst` | known_results + conjecture | `research_gap` |
| 3 | `ProofArchitect` | research_gap + known_results | `proof_plan[]` (provenance-annotated) |
| 4 | `LemmaDeveloper` | proof_plan, open_goals | `proven_lemmas{}` |
| 5 | `Assembler` | proven_lemmas | `assembled_proof` |
| 6 | `TheoremCrystallizer` | assembled_proof | `formal_statement` |
| 7 | `ConsistencyChecker` | full TheoryState | consistency report |

**LemmaDeveloper inner loop** (per lemma):
```
for each open_goal:
    Prover → Verifier → (if failed) Refiner → repeat
    CounterexampleSearcher runs in parallel
    Stagnation detection: if same error N times → force Refiner
```

**Provenance system:** Each lemma in the proof plan is annotated as `known` (directly citable), `adapted` (needs modification), or `new` (must be fully proved). Only `adapted` and `new` lemmas enter the proof loop.

**Auto-verify:** Proofs with confidence ≥ `AUTO_VERIFY_CONFIDENCE` (default 0.95) are accepted without an LLM verifier call. The LLM Verifier itself uses a separate pass threshold `VERIFIER_PASS_CONFIDENCE` (default 0.90).

**ProofArchitect retry policy:** If the full provenance-annotated plan fails (e.g. the LLM returns a field as `null`), the architect retries with a simplified 3-lemma prompt (foundational → central bound → main result). Only if both attempts fail does it fall back to a single `main_result` goal.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EurekaClaw/EurekaClaw](https://github.com/EurekaClaw/EurekaClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
