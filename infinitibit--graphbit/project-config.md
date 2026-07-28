---
trigger: always_on
description: Agents are AI-powered components that execute tasks within GraphBit workflows. This guide covers how to create, configure, and optimize agents for different use cases.
---

# Agents

Agents are AI-powered components that execute tasks within GraphBit workflows. This guide covers how to create, configure, and optimize agents for different use cases.

## Overview

In GraphBit, agents are implemented as specialized workflow nodes that:
- Execute AI tasks using configured LLM providers
- Process inputs through prompt templates with variable substitution
- Generate outputs that flow to connected nodes
- Support different execution contexts and requirements

## Creating Agents

### Basic Agent Creation

```python
from graphbit import Node

# Create a basic agent node
analyzer = Node.agent(
    name="Data Analyzer",
    prompt=f"Analyze the following data and identify key patterns: {input}",
    agent_id="analyzer",  # Optional - auto-generated if not provided
    temperature=0.7,  # Optional - controls randomness (0.0 to 2.0)
    max_tokens=1000  # Optional - maximum tokens in response
)

# Access agent properties
print(f"Agent ID: {analyzer.id()}")
print(f"Agent Name: {analyzer.name()}")
```

### Agent with Explicit Configuration

```python
# Agent with explicit ID for referencing
content_creator = Node.agent(
    name="Content Creator",
    prompt=f"Create engaging content about: {topic}",
    agent_id="content_creator_v1"
)

# Agent for specific domain
technical_writer = Node.agent(
    name="Technical Documentation Writer",
    prompt=f"""
    Write comprehensive technical documentation for: {feature}
    
    Include:
    - Overview and purpose
    - Implementation details
    - Usage examples
    - Best practices
    
    Feature details: {input}
    """,
    agent_id="tech_doc_writer"
)
```

## Agent Configuration in Workflows

### Single Agent Workflow

```python
from graphbit import Workflow, Node, LlmConfig, Executor

# Create workflow with single agent
workflow = Workflow("Content Analysis")

# Create and add agent
analyzer = Node.agent(
    name="Content Analyzer",
    prompt=f"Analyze this content for sentiment, key themes, and quality: {input}",
    agent_id="content_analyzer"
)

analyzer_id = workflow.add_node(analyzer)
workflow.validate()

# Execute with LLM configuration
llm_config = LlmConfig.openai(
    api_key=os.getenv("OPENAI_API_KEY"),
    model="gpt-4o-mini"
)

executor = Executor(llm_config, timeout_seconds=60)
result = executor.execute(workflow)
```

### Multi-Agent Workflow

```python
from graphbit import Workflow, Node

# Create workflow with multiple specialized agents
workflow = Workflow("Multi-Agent Analysis Pipeline")

# Create specialized agents
sentiment_agent = Node.agent(
    name="Sentiment Analyzer",
    prompt=f"Analyze the sentiment of this text (positive/negative/neutral): {input}",
    agent_id="sentiment_analyzer"
)

topic_agent = Node.agent(
    name="Topic Extractor", 
    prompt=f"Extract the main topics and themes from: {input}",
    agent_id="topic_extractor"
)

summary_agent = Node.agent(
    name="Content Summarizer",
    prompt=f"Create a concise summary of: {input}",
    agent_id="summarizer"
)

# Aggregation agent
aggregator = Node.agent(
    name="Analysis Aggregator",
    prompt="""
    Combine the following analysis results into a comprehensive report:
    
    Sentiment Analysis, Topic Analysis and Summary.
    
    Provide an integrated analysis with key insights.
    """,
    agent_id="aggregator"
)

# Build workflow
sentiment_id = workflow.add_node(sentiment_agent)
topic_id = workflow.add_node(topic_agent)
summary_id = workflow.add_node(summary_agent)
agg_id = workflow.add_node(aggregator)

# Connect nodes for parallel processing then aggregation
workflow.connect(sentiment_id, agg_id)
workflow.connect(topic_id, agg_id)
workflow.connect(summary_id, agg_id)

workflow.validate()
```

## Prompt Engineering

### Basic Prompt Structure

Design effective prompts for your agents:

```python
# Simple, direct prompt
simple_agent = Node.agent(
    name="Simple Translator",
    prompt=f"Translate this text to French: {input}",
    agent_id="translator"
)

# Structured prompt with clear instructions
structured_agent = Node.agent(
    name="Structured Analyzer",
    prompt=f"""
    Task: Analyze the provided text for business insights
    
    Text to analyze: {input}
    
    Please provide:
    1. Key business themes identified
    2. Market opportunities mentioned
    3. Risk factors highlighted  
    4. Recommended actions
    
    Format your response as a structured analysis.
    """,
    agent_id="business_analyzer"
)
```

### Variable Substitution

Use variables in prompts for dynamic content:

```python
# Multi-variable prompt
flexible_prompt = f"""
Context: You are a {role} expert analyzing {content_type} content.

Task: {task_description}

Content to analyze: {input}

Analysis requirements:
- Focus on {focus_area}
- Provide {detail_level} analysis
- Use {tone} tone
- Consider {constraints}

Please provide your analysis following these requirements.
"""

flexible_agent = Node.agent(
    name="Flexible Content Analyzer",
    prompt=flexible_prompt,
    agent_id="flexible_analyzer"
)
```

### Domain-Specific Prompts

Create agents for specific domains:

```python
# Financial analysis agent

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [InfinitiBit/graphbit](https://github.com/InfinitiBit/graphbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
