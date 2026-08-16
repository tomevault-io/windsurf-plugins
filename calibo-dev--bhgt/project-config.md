---
trigger: always_on
description: This project implements a multi-agent system using CrewAI framework to research and report on various topics. The system consists of two specialized agents working in a sequential process to gather information and generate comprehensive reports.
---

# Agents Documentation

## Overview

This project implements a multi-agent system using CrewAI framework to research and report on various topics. The system consists of two specialized agents working in a sequential process to gather information and generate comprehensive reports.

## Architecture

The agent system is built on CrewAI 1.8.1 and uses:
- **LLM Provider Selection**: Controlled by `PROVIDER` environment variable
  - `PROVIDER=OPENAI`: Uses OpenAI GPT-4.1-mini (requires API key secret retrieval)
  - `PROVIDER=ANTHROPICAI`: Uses Anthropic models (requires API key secret retrieval)
  - `PROVIDER=AZUREAI`: Uses Azure models (requires API key secret retrieval)
  - `PROVIDER=GEMINIAI`: Uses Gemini models (requires API key secret retrieval)
  - `PROVIDER=OLLAMA` (default): Uses Ollama with llama3.2 model
- **Process Type**: Sequential
- **Framework**: CrewAI with FastAPI integration
- **Python Version**: 3.10 - 3.13

## Agents

### 1. Researcher Agent

**Role**: `{topic} Senior Data Researcher`

**Goal**: Uncover cutting-edge developments in the specified topic

**Backstory**: 
A seasoned researcher with a knack for uncovering the latest developments in any given topic. Known for the ability to find the most relevant information and present it in a clear and concise manner.

**Configuration**:
- LLM: `ollama/llama3.2`
- Temperature: 0.7
- Verbose: True

**Primary Task**: Research Task
- Conducts thorough research about the specified topic
- Finds interesting and relevant information for the current year
- Outputs: A list with 10 bullet points of the most relevant information

### 2. Reporting Analyst Agent

**Role**: `{topic} Reporting Analyst`

**Goal**: Create detailed reports based on data analysis and research findings

**Backstory**: 
A meticulous analyst with a keen eye for detail. Known for the ability to turn complex data into clear and concise reports, making it easy for others to understand and act on the information provided.

**Configuration**:
- LLM: `ollama/llama3.2`
- Temperature: 0.7
- Verbose: True

**Primary Task**: Reporting Task
- Reviews the research context and expands each topic into a full section
- Creates detailed reports containing all relevant information
- Outputs: A fully-fledged markdown report with main topics and detailed sections
- Output File: `report.md`

## Workflow

The agents work in a sequential process:

```
1. Researcher Agent
   └─> Conducts research on the topic
       └─> Generates 10 bullet points of key findings
           └─> Passes context to next agent

2. Reporting Analyst Agent
   └─> Reviews research findings
       └─> Expands each point into detailed sections
           └─> Generates final markdown report (report.md)
```

## Configuration Files

### agents.yaml
Located at: `src/latest_ai_development/config/agents.yaml`

Defines agent roles, goals, backstories, and LLM configurations.

### tasks.yaml
Located at: `src/latest_ai_development/config/tasks.yaml`

Defines task descriptions, expected outputs, and agent assignments.

## Usage

### Running the Crew

The system can be executed in multiple ways:

#### 1. Direct Execution
```python
from latest_ai_development.crew import LatestAiDevelopment
from datetime import datetime

inputs = {
    'topic': 'Your Topic Here',
    'current_year': str(datetime.now().year)
}

LatestAiDevelopment().crew().kickoff(inputs=inputs)
```

#### 2. FastAPI Endpoint
```bash
POST /ask
{
    "topic": "Your Topic Here"
}
```

#### 3. Command Line
```bash
# Run the crew
latest_ai_development

# Train the crew
train <n_iterations> <filename>

# Replay a specific task
replay <task_id>

# Test the crew
test <n_iterations> <eval_llm>

# Run with trigger payload
run_with_trigger '<json_payload>'
```

## Environment Variables

- `PROVIDER`: LLM provider selection (OPENAI, ANTHROPICAI, AZURE, GEMINIAI, or OLLAMA, default: OLLAMA)
- `CLOUD_PROVIDER`: Secret backend selection (`AWS` or `AZURE`)
- `API_KEY_SECRET`: Preferred secret name used to retrieve the provider API key
- `OPENAI_API_KEY_SECRET`: Backward-compatible fallback secret name for OpenAI when `API_KEY_SECRET` is not set
- `ANTHROPICAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Anthropic when `API_KEY_SECRET` is not set
- `AZUREAI_OPENAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Azure OpenAI when `API_KEY_SECRET` is not set
- `GEMINIAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Gemini when `API_KEY_SECRET` is not set
- `OLLAMA_BASE_URL`: Base URL for Ollama API (used when PROVIDER=OLLAMA)
- `TRACING_BACKEND`: Tracing backend selector (`LANGSMITH`, `CREWAI`, or `NONE`). When unset or blank, the app falls back to `CREWAI`.
- `LANGSMITH_ENDPOINT`: LangSmith API endpoint (for example `https://api.smith.langchain.com`)
- `LANGSMITH_PROJECT`: LangSmith project name for trace grouping
- `LANGSMITH_API_KEY`: Direct LangSmith API key value
- `LANGSMITH_API_KEY_SECRET`: Preferred secret name used to resolve the LangSmith API key through the configured secret manager
- `PORT`: Server port (default: 8080)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calibo-dev/bhgt](https://github.com/calibo-dev/bhgt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
