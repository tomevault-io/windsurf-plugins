---
trigger: always_on
description: You are an expert in Python, Mirascope, and the Sygaldry AI framework.
---

# Sygaldry Development Rules

You are an expert in Python, Mirascope, and the Sygaldry AI framework.

## Core Principles

- Write clean, maintainable Python code following PEP 8
- Use Mirascope's functional patterns with `@prompt_template` decorators
- Implement Pydantic models for all LLM responses
- Prefer async/await patterns for optimal performance
- Follow the component structure defined in component.json files

## Mirascope Best Practices

- Always use `@prompt_template` decorators for prompt construction
- Define Pydantic response models for structured outputs
- Use async functions for all LLM calls and tools
- Implement proper error handling and validation
- Include comprehensive docstrings and type hints

## Component Development

- Follow the sygaldry component structure with proper JSON manifests
- Implement registry dependencies correctly
- Include comprehensive examples and documentation
- Test all functionality before committing
- Support multiple LLM providers when possible

## Available Components

- **text_summarization_agent** (agent): Advanced text summarization agent using chain-of-thought reasoning, few-shot learning, and iterative refinement. Supports multiple styles (technical, executive, simple, academic, journalistic) and progressive summarization with validation.
- **multi_source_news_verification** (agent): Advanced multi-source news verification agent with comprehensive fact-checking tools including academic search, government data verification, social media verification, and expert source validation for combating misinformation
- **multi_agent_coordinator** (agent): Orchestrates multiple specialized agents to solve complex tasks through intelligent task decomposition, agent selection, and result synthesis
- **recruiting_assistant_agent** (agent): Recruiting assistant for finding qualified candidates using Exa websets. Helps with technical recruiting, sales hiring, and executive search.
- **game_theory_analysis** (agent): Analyzes complex strategic situations using game theory principles, identifying equilibria, predicting outcomes, and providing actionable recommendations
- **enhanced_knowledge_graph_agent** (agent): Enhanced knowledge graph extraction using advanced prompt engineering. Features meta-reasoning for strategy planning, chain-of-thought entity extraction with detailed reasoning, multi-pass relationship detection, and self-consistency validation for high-accuracy results.
- **document_segmentation_agent** (agent): Agent for intelligently segmenting documents into logical parts. Supports multiple strategies including semantic, structural, hybrid, and fixed-size segmentation. Features document structure analysis, segment summarization, and optimized chunking for vector embeddings.
- **knowledge_graph_agent** (agent): Agent for extracting structured knowledge from text by identifying entities and their relationships. Builds comprehensive knowledge graph representations with support for hierarchical relationships, graph enrichment, and visualization-ready outputs.
- **prompt_engineering_optimizer** (agent): Advanced prompt optimization agent that analyzes, generates variants, performs A/B testing, and delivers production-ready optimized prompts with comprehensive documentation
- **academic_research_agent** (agent): Academic research agent for finding research papers using Exa websets. Perfect for academics, researchers, and anyone needing to discover scholarly publications.
- **game_playing_catan** (agent): Multi-model turn-based Settlers of Catan game agent supporting AI vs AI, human vs AI, or mixed gameplay with resource management, trading, and strategic building
- **research_assistant_agent** (agent): AI-powered research agent that conducts comprehensive research using Exa search
- **pii_scrubbing_agent** (agent): Agent for detecting and removing Personally Identifiable Information (PII) from text. Combines regex patterns and LLM analysis for comprehensive PII detection. Supports multiple scrubbing methods including masking, redaction, generalization, and synthetic data replacement.
- **dataset_builder_agent** (agent): AI-powered dataset builder that creates curated data collections using Exa Websets with custom criteria and enrichments
- **dnd_game_master** (agent): A comprehensive D&D 5e game master agent with full rules enforcement and persistent campaign state. Features SQLite-based state persistence for multi-session campaigns, fair dice rolling with modifiers, complete D&D 5e API integration, multi-model orchestration, turn-based combat with positioning, spell slot tracking, condition management, death saves, XP/leveling, exhaustion, skill proficiencies, inventory management, and dynamic roleplay with human-in-the-loop support.
- **multi_platform_social_media_manager** (agent): Enhanced multi-platform social media campaign manager with trend analysis, engagement prediction, and real-time adaptation capabilities for comprehensive campaign orchestration
- **decision_quality_assessor** (agent): Comprehensive decision quality assessment agent that analyzes context, evaluates alternatives, detects cognitive biases, and provides actionable recommendations for better decision-making

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greyhaven-ai/sygaldry](https://github.com/greyhaven-ai/sygaldry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
