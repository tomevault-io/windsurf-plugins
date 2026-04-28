---
trigger: always_on
description: > Documentation for AI agents used in sklearn-diagnose
---

# AGENTS.md

> Documentation for AI agents used in sklearn-diagnose

## Overview

sklearn-diagnose uses a **multi-agent architecture** powered by [LangChain](https://langchain.com/) to diagnose machine learning model failures. The system employs four specialized AI agents: three for batch diagnosis and one for interactive conversation.

```
┌─────────────────────────────────────────────────────────────────────┐
│                     sklearn-diagnose Agent System                    │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   Signal Extraction (Deterministic)                                  │
│   ─────────────────────────────────                                  │
│   • Train/validation scores                                          │
│   • Cross-validation metrics                                         │
│   • Feature correlations                                             │
│   • Class distributions                                              │
│                         │                                            │
│                         ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                   HYPOTHESIS AGENT                           │   │
│   │   Analyzes signals → Detects failure modes                   │   │
│   │   Output: List of hypotheses with confidence & evidence      │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                         │                                            │
│                         ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                 RECOMMENDATION AGENT                         │   │
│   │   Analyzes hypotheses → Generates actionable fixes           │   │
│   │   Output: Prioritized list of recommendations                │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                         │                                            │
│                         ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    SUMMARY AGENT                             │   │
│   │   Synthesizes findings → Creates human-readable report       │   │
│   │   Output: Markdown-formatted diagnostic summary              │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                         │                                            │
│                         ▼                                            │
│                  DiagnosisReport                                     │
│                         │                                            │
│                         ▼                                            │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                     CHAT AGENT (Interactive)                 │   │
│   │   User conversations → Context-aware Q&A                     │   │
│   │   Output: Interactive responses with conversation history    │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Agent Specifications

### 1. Hypothesis Agent

**Purpose**: Analyze model performance signals and identify potential failure modes.

**Role**: Expert ML diagnostician that detects issues like overfitting, underfitting, data leakage, class imbalance, and more.

| Property | Value |
|----------|-------|
| **Location** | `sklearn_diagnose/llm/client.py` |
| **Method** | `LangChainClient.generate_hypotheses()` |
| **System Prompt** | `HYPOTHESIS_SYSTEM_PROMPT` |
| **Input** | Extracted signals (dict), Task type (str) |
| **Output** | List of `Hypothesis` objects |

#### System Prompt

```
You are an expert ML diagnostician agent. Your task is to analyze model 
performance signals and identify potential failure modes.

You will be given:
1. Performance metrics (train score, validation score, CV scores, etc.)
2. A list of possible failure modes to consider

For each failure mode you detect, you must provide:
- failure_mode: The name of the failure mode (must be one of the provided options)
- confidence: A score between 0.0 and 1.0 indicating how confident you are (0.95 max)
- severity: "low", "medium", or "high"
- evidence: A list of specific observations that support this hypothesis

Guidelines:
- Only report failure modes with confidence >= 0.25
- Be conservative - don't over-diagnose
- Base your assessment solely on the provided signals
- Provide specific, quantitative evidence when possible
- A model can have multiple failure modes simultaneously
```

#### Output Schema

```json
{
  "hypotheses": [
    {
      "failure_mode": "overfitting",
      "confidence": 0.85,
      "severity": "high",
      "evidence": [
        "Train-val gap of 25% indicates memorization",
        "Perfect training score (100%) with 75% validation"
      ]
    }
  ]
}
```

#### Detectable Failure Modes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leockl/sklearn-diagnose](https://github.com/leockl/sklearn-diagnose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
