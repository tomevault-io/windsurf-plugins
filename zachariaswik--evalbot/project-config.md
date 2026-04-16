---
trigger: always_on
description: EvalBot is a CrewAI-based system for evaluating early-stage startup submissions through a multi-agent pipeline. Each agent specializes in one aspect of venture analysis (intake parsing, venture scoring, market analysis, etc.) and produces structured JSON outputs that feed into downstream agents.
---

# EvalBot — Multi-Agent Startup Evaluation Pipeline

EvalBot is a CrewAI-based system for evaluating early-stage startup submissions through a multi-agent pipeline. Each agent specializes in one aspect of venture analysis (intake parsing, venture scoring, market analysis, etc.) and produces structured JSON outputs that feed into downstream agents.

## Running the System

### Setup
```bash
# Use Python 3.13 or earlier (CrewAI stack incompatible with 3.14+)
python -m venv .venv313
source .venv313/bin/activate  # or .venv313\Scripts\activate on Windows

pip install -r requirements.txt  # if it exists, otherwise install crewai, litellm, pydantic

# Set up API keys in .env
ANTHROPIC_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
MINIMAX_API_KEY=your_key_here
```

### CLI Commands
```bash
# Process default CourseDocs submission
python main.py

# Process a single startup submission (text, PDF, or DOCX)
python main.py single <path/to/submission.txt>
python main.py single <path/to/pitch.pdf>
python main.py single <path/to/deck.docx>

# Process a batch of startups from a directory
python main.py batch <path/to/startups_folder>

# Generate synthetic startup ideas and evaluate them
python main.py generate --rounds 3 --screening-threshold 50 --max-attempts 5
```

## Architecture

### Multi-Agent Pipeline
The system uses **8 specialized agents** that run sequentially:

0. **Startup Idea Generator** (Agent 0) — Generate mode only
   - Creates synthetic startup submissions optimized for scoring well
   - Only runs in `generate` mode with screening loop

1. **Intake Parser** (Agent 1) — Low reasoning complexity
   - Converts raw submissions into standardized JSON structure
   - Extracts: problem, solution, market, team, competitors, traction, etc.
   - Flags missing/inconsistent information
   - **PDF Support**: Can read PDF documents directly (uses Claude Sonnet 4.6's native PDF vision)
   - **DOCX Support**: Extracts text and tables (metrics, financials, competitive data)
   - No PDF pre-processing needed — PDFs passed via CrewAI's `input_files` parameter
   - DOCX tables containing TAM, revenue, team size, etc. are properly extracted


2. **Venture Analyst** (Agent 2) — Very high reasoning complexity
   - Core scoring agent: evaluates 10 dimensions (1-10 scale each)
   - Produces verdict: Top VC Candidate, Promising (Needs Focus/Pivot), Good Small Business, Feature Not Company, AI Wrapper, Reject
   - Outputs SWOT analysis and weighted total score
   - In `generate` mode, runs in screening loop with Agent 0 & 1

3. **Market & Competition Analyst** (Agent 3) — Medium-high complexity
   - Market dynamics, TAM/SAM/SOM analysis
   - Competitive landscape assessment

4. **Product & Positioning Analyst** (Agent 4) — Medium complexity
   - Feature vs company distinction
   - AI wrapper risk assessment
   - Product positioning

5. **Founder Fit Analyst** (Agent 5) — Medium complexity
   - Team assessment
   - Execution confidence

6. **Recommendation / Pivot Agent** (Agent 6) — Very high complexity
   - Strategic synthesis of all prior analyses
   - Actionable recommendations and pivot options
   - Most important output for users

7. **Ranking Committee** (Agent 7) — Medium complexity
   - Only runs in batch mode
   - Comparative ranking across cohort
   - Input context grows with batch size

### Agent Structure
- Each agent lives in `Agent{N}/` directory with a `prompt.md` file
- Prompts are loaded at runtime by `src/agents.py`
- Agent metadata (role, goal) defined in `_AGENT_META` dict in `src/agents.py`
- Pydantic output models defined in `src/models.py` (one per agent)

### Feedback Loop / Re-run System
Agents 2-6 can request re-runs from earlier agents via `FeedbackMixin` fields:
- `rerun_from_agent`: int (which agent to re-run from)
- `rerun_reason`: str (explanation)

The pipeline enforces a `MAX_ITERATIONS` safety limit (default: 18 = 3x the 6-agent loop).

### Generate Mode — Screening Loop
When running `python main.py generate`, the system:
1. Agent 0 generates a startup idea optimized to score well
2. Agent 1 parses it
3. Agent 2 scores it
4. If score < threshold, regenerate with feedback (up to `--max-attempts` times per round)
5. Best attempt proceeds to full evaluation (Agents 3-6)
6. Repeat for `--rounds` number of startups
7. Agent 7 ranks the cohort

## Configuration

### LLM Model Assignment (`src/config.py`)
- CrewAI uses **litellm** under the hood — any litellm-compatible model string works
- `DEFAULT_MODEL`: Fallback model (currently `anthropic/claude-haiku-4-5`)
- `AGENT_MODELS`: Dict mapping agent number → model string override
- `RERUN_MODEL`: Model used for feedback-triggered re-runs (None = reuse agent's normal model)
- `get_model_for_agent(agent_number, is_rerun)`: Resolves model priority

See `agent_models.md` for detailed model recommendations by provider and cost profiles.

**Supported providers** (set API key in `.env`):
- OpenAI: `gpt-4o`, `gpt-4.1`, `gpt-4.1-mini`, `gpt-4.1-nano`, `o3-mini`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zachariaswik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
