---
trigger: always_on
description: This repository contains "AI Agents for Beginners" - a comprehensive educational course teaching everything needed to build AI Agents. The course consists of 18 lessons (numbered 00-18) covering fundamentals, design patterns, frameworks, production deployment, local/on-device agents, and security of AI agents.
---

# AGENTS.md

## Project Overview

This repository contains "AI Agents for Beginners" - a comprehensive educational course teaching everything needed to build AI Agents. The course consists of 18 lessons (numbered 00-18) covering fundamentals, design patterns, frameworks, production deployment, local/on-device agents, and security of AI agents.

**Key Technologies:**
- Python 3.12+
- Jupyter Notebooks for interactive learning
- AI Frameworks: Microsoft Agent Framework (MAF)
- Azure AI Services: Microsoft Foundry, Microsoft Foundry Agent Service V2

**Architecture:**
- Lesson-based structure (00-18 directories)
- Each lesson contains: README documentation, code samples (Jupyter notebooks), and images
- Multi-language support via automated translation system
- One Python notebook per lesson using Microsoft Agent Framework

## Setup Commands

### Prerequisites
- Python 3.12 or higher
- Azure subscription (for Microsoft Foundry)
- Azure CLI installed and authenticated (`az login`)

### Initial Setup

1. **Clone or fork the repository:**
   ```bash
   gh repo fork microsoft/ai-agents-for-beginners --clone
   # OR
   git clone https://github.com/microsoft/ai-agents-for-beginners.git
   cd ai-agents-for-beginners
   ```

2. **Create and activate Python virtual environment:**
   ```bash
   python3 -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

4. **Set up environment variables:**
   ```bash
   cp .env.example .env
   # Edit .env with your API keys and endpoints
   ```

### Required Environment Variables

For **Microsoft Foundry** (Required):
- `AZURE_AI_PROJECT_ENDPOINT` - Microsoft Foundry project endpoint
- `AZURE_AI_MODEL_DEPLOYMENT_NAME` - Model deployment name (e.g., gpt-5-mini)

For **Azure AI Search** (Lesson 05 - RAG):
- `AZURE_SEARCH_SERVICE_ENDPOINT` - Azure AI Search endpoint
- `AZURE_SEARCH_API_KEY` - Azure AI Search API key

Authentication: Run `az login` before running notebooks (uses `AzureCliCredential`).

## Development Workflow

### Running Jupyter Notebooks

Each lesson contains multiple Jupyter notebooks for different frameworks:

1. **Start Jupyter:**
   ```bash
   jupyter notebook
   ```

2. **Navigate to a lesson directory** (e.g., `01-intro-to-ai-agents/code_samples/`)

3. **Open and run notebooks:**
   - `*-python-agent-framework.ipynb` - Using Microsoft Agent Framework (Python)
   - `*-dotnet-agent-framework.ipynb` - Using Microsoft Agent Framework (.NET)

### Working with Microsoft Agent Framework

**Microsoft Agent Framework + Microsoft Foundry:**
- Requires Azure subscription
- Uses `FoundryChatClient` for Agent Service V2 (agents visible in Foundry portal)
- Production-ready with built-in observability
- File pattern: `*-python-agent-framework.ipynb`

## Testing Instructions

This is an educational repository with example code rather than production code with automated tests. To verify your setup and changes:

### Manual Testing

1. **Test Python environment:**
   ```bash
   python --version  # Should be 3.12+
   pip list | grep -E "(agent-framework|azure-ai|azure-identity)"
   ```

2. **Test notebook execution:**
   ```bash
   # Convert notebook to script and run (tests imports)
   jupyter nbconvert --to script <lesson-folder>/code_samples/<notebook>.ipynb --stdout | python
   ```

3. **Verify environment variables:**
   ```bash
   python -c "import os; from dotenv import load_dotenv; load_dotenv(); print('✓ AZURE_AI_PROJECT_ENDPOINT' if os.getenv('AZURE_AI_PROJECT_ENDPOINT') else '✗ AZURE_AI_PROJECT_ENDPOINT missing')"
   ```

### Running Individual Notebooks

Open notebooks in Jupyter and execute cells sequentially. Each notebook is self-contained and includes:
- Import statements
- Configuration loading
- Example agent implementations
- Expected outputs in markdown cells

### Smoke-Testing Deployed Agents

For lessons where an agent is deployed as a Microsoft Foundry hosted agent (01, 04, 05, 16), the repo ships smoke-test catalogs under `tests/` that are run by the `.github/workflows/smoke-test.yml` workflow via the [AI Smoke Test](https://github.com/marketplace/actions/ai-smoke-test) action. These are a lightweight post-deploy gate (is the agent reachable and following basic prompt expectations?), complementing the evaluation pipeline in Lessons 10 and 16. See [tests/README.md](./tests/README.md) for the catalog-to-lesson-to-agent mapping. Lesson 17 runs locally with Foundry Local and has no hosted endpoint, so it is validated by running its notebook directly.

## Code Style

### Python Conventions

- **Python Version**: 3.12+
- **Code Style**: Follow standard Python PEP 8 conventions
- **Notebooks**: Use clear markdown cells to explain concepts
- **Imports**: Group by standard library, third-party, local imports

### Jupyter Notebook Conventions

- Include descriptive markdown cells before code cells
- Add output examples in notebooks for reference
- Use clear variable names that match lesson concepts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoltEmperorChaos/ai-agent-for-pc](https://github.com/VoltEmperorChaos/ai-agent-for-pc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
