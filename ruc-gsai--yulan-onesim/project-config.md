---
trigger: always_on
description: YuLan-OneSim implements a sophisticated multi-agent system where each agent is an autonomous entity powered by LLMs. The agent system is built on a modular architecture with four core components working together to create intelligent, reactive behaviors.
---

# Agent System

YuLan-OneSim implements a sophisticated multi-agent system where each agent is an autonomous entity powered by LLMs. The agent system is built on a modular architecture with four core components working together to create intelligent, reactive behaviors.

## Agent Architecture Overview

Each agent in YuLan-OneSim consists of four fundamental modules:

- **Profile Module**: Defines agent identity, attributes, and characteristics
- **Memory Module**: Manages memory with retrieval and storage
- **Planning Module**: Implements decision-making strategies and goal planning
- **Action Module**: Executes planned behaviors and triggers simulation events


## Profile Module

The Profile module defines agent identity and characteristics through a structured attribute system.

### Attribute Structure
```json
{
  "public_attributes": {
    "name": "Alice Johnson",
    "age": 28,
    "occupation": "Data Scientist",
    "location": "San Francisco"
  },
  "private_attributes": {
    "personality": "analytical, introverted",
    "risk_tolerance": "moderate",
    "political_leaning": "liberal",
    "income_level": "high"
  }
}
```

### Key Features
- **Public Attributes**: Visible to other agents for social interaction
- **Private Attributes**: Hidden characteristics that influence decision-making
- **Dynamic Evolution**: Attributes can change based on experiences and interactions
- **Relationship Tracking**: Maintains connections and relationship states with other agents

## Memory Module

The Memory module implements a sophisticated memory management system that mimics human memory processes.

### Memory Architecture
```json
{
  "strategy": "ShortLongStrategy",
  "storages": {
      "short_term_storage": {
          "class": "ListMemoryStorage",
          "capacity": 100
      },
      "long_term_storage": {
          "class": "VectorMemoryStorage",
          "capacity": 100,
          "model_config_name": "openai_embedding-bert"
      }
  },
  "operations": {
      "add": {
          "class": "AddMemoryOperation"
      },
      "retrieve": {
          "class": "RetrieveMemoryOperation"
      },
      "remove": {
          "class": "RemoveMemoryOperation"
      }
  },
  "metrics": {
      "recency": {
          "class": "RecencyMetric",
          "weight": 0.5
      },
      "relevance": {
          "class": "RelevanceMetric",
          "model_config_name": "openai_embedding-bert",
          "weight": 0.5
      }
  }
}
```

### Memory Strategies

#### ShortLongStrategy
- **Short-term Memory**: Recent experiences stored in chronological order
- **Long-term Memory**: Important experiences stored with vector embeddings
- **Transfer Mechanism**: Automatic promotion from short-term to long-term based on metrics

#### Storage Types

**ListMemoryStorage**
- Simple chronological storage
- Fast insertion and retrieval
- Suitable for recent memory access

**VectorMemoryStorage**  
- Semantic similarity-based storage
- Uses embedding models for content representation
- Enables contextual memory retrieval

### Memory Retrieval
Memory retrieval considers three factors:
- **Recency**: How recently the memory was formed
- **Importance**: Significance of the memory content
- **Relevance**: Semantic similarity to current context

## Planning Module

The Planning module implements different cognitive architectures for agent decision-making.

### Planning Algorithms

#### COT (Chain-of-Thought) Planning
```python
class COTPlanning(PlanningBase):
    """Single-step reasoning for immediate decisions"""

    async def plan(self,**kwargs) -> str:
        prompt=f"""
        ### Agent Profile
        {kwargs["profile"]}

        ### Memory
        {kwargs["memory"]}

        
        ### Observation
        {kwargs["observation"]}
        
        ### Instruction
        {kwargs["instruction"]}

        Please think step by step based on the above concisely.
        """
        prompt=self.model.format(
            Message("system", self.sys_prompt, role="system"),
            Message("user", prompt, role="user")
        )
        response = await self.model.acall(prompt)
        return response.text

```

- **Use Case**: Simple, reactive decisions
- **Characteristics**: Fast, direct reasoning
- **Best For**: Immediate responses, simple interactions

#### BDI (Belief-Desire-Intention) Planning
```python
class BDIPlanning(PlanningBase):
    """Goal-oriented planning with beliefs and intentions"""
    
    async def plan(self, **kwargs) -> str:
        prompt = f"""
        ### Agent Profile
        {kwargs["profile"]}

        ### Memory (Beliefs)
        {kwargs["memory"]}
        
        ### Observation (New Beliefs)
        {kwargs["observation"]}
        
        ### Instruction (Task)
        {kwargs["instruction"]}

        Please analyze the situation using the BDI (Belief-Desire-Intention) framework:
        
        1. Beliefs: Based on the agent's memory and current observations, what does the agent believe about the current state of the world?
        

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RUC-GSAI/YuLan-OneSim](https://github.com/RUC-GSAI/YuLan-OneSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
