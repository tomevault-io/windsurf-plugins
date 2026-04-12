---
trigger: always_on
description: Comprehensive guide for creating, running, and debugging scenario tests in ElizaOS
---

# ElizaOS Scenario Testing

This guide explains how to create, run, and debug scenario tests for ElizaOS agents, plugins, and complex workflows using the integrated scenario testing framework.

## Overview

ElizaOS scenario tests are **real-world agent simulations** that:

- Create multi-agent conversations with defined roles and scripts
- Test complex workflows and plugin integrations
- Verify agent behaviors in realistic scenarios
- Use actual AI models, databases, and external services
- Provide LLM-based verification of outcomes
- Support concurrent execution and performance benchmarking

**A scenario should be created to demonstrate every single feature running in a production-like environment.**

## Core Architecture

### Scenario Structure

```typescript
interface Scenario {
  id: UUID;                    // Unique identifier
  name: string;               // Human-readable name
  description: string;        // Purpose and context
  category: string;           // Classification (integration, unit, stress)
  tags: string[];             // Searchable tags
  actors: ScenarioActor[];    // Participating agents
  setup?: ScenarioSetup;      // Environment configuration
  execution?: ScenarioExecution; // Runtime parameters
  verification: ScenarioVerification; // Success criteria
}
```

### Actor Types

```typescript
interface ScenarioActor {
  id: UUID;
  name: string;
  role: 'subject' | 'observer' | 'assistant' | 'adversary';
  bio?: string;               // Agent personality
  system?: string;            // System prompt
  plugins?: string[];         // Required plugins
  script?: ScenarioScript;    // Scripted behaviors
}

// Role Definitions:
// - subject: Primary agent being tested
// - observer: Passive monitoring agent
// - assistant: Helpful secondary agent
// - adversary: Creates challenging scenarios
```

## Creating Scenario Tests

### 1. Basic Scenario Template

```typescript
import type { Scenario } from '../../src/scenario-runner/types.js';

export const myScenario: Scenario = {
  id: 'uuid-here',
  name: 'My Test Scenario',
  description: 'Tests specific agent functionality',
  category: 'integration',
  tags: ['plugin-name', 'feature-area'],

  actors: [
    {
      id: 'subject-uuid',
      name: 'Test Agent',
      role: 'subject',
      bio: 'Agent description',
      system: 'System prompt for the agent',
      plugins: ['@elizaos/plugin-required'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'Hello, I want to test something specific.',
            description: 'Initial test message'
          }
        
      }
    }
  ],

  verification: {
    rules: [
      {
        id: 'response-check',
        type: 'llm',
        description: 'Verify agent responds appropriately',
        config: {
          successCriteria: 'Agent should acknowledge the request and take appropriate action',
          priority: 'high'
        }
      }
    
  }
};
```

### 2. Multi-Agent Scenario

```typescript
export const multiAgentScenario: Scenario = {
  id: 'multi-agent-uuid',
  name: 'Multi-Agent Collaboration',
  description: 'Tests agents working together',
  category: 'integration',
  tags: ['collaboration', 'multi-agent'],

  actors: [
    {
      id: 'agent1-uuid',
      name: 'Primary Agent',
      role: 'subject',
      plugins: ['@elizaos/plugin-rolodex'],
      script: { steps: [] }
    },
    {
      id: 'agent2-uuid', 
      name: 'Helper Agent',
      role: 'assistant',
      plugins: ['@elizaos/plugin-research'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'I can help with research tasks.',
            description: 'Offer assistance'
          },
          {
            type: 'wait',
            waitTime: 2000,
            description: 'Wait for response'
          },
          {
            type: 'message', 
            content: 'What would you like me to look up?',
            description: 'Follow-up query'
          }
        
      }
    }
  ],

  execution: {
    maxDuration: 30000,
    maxSteps: 20
  },

  verification: {
    rules: [
      {
        id: 'collaboration-check',
        type: 'llm',
        description: 'Verify agents collaborate effectively',
        config: {
          successCriteria: 'Agents should work together, share information, and complete the task',
          category: 'collaboration'
        }
      }
    
  }
};
```

### 3. Plugin Testing Scenario

```typescript
export const pluginTestScenario: Scenario = {
  id: 'plugin-test-uuid',
  name: 'Plugin Functionality Test',
  description: 'Tests specific plugin actions and behaviors',
  category: 'integration',
  tags: ['plugin-test', 'actions'],

  actors: [
    {
      id: 'plugin-agent-uuid',
      name: 'Plugin Test Agent',
      role: 'subject',
      bio: 'Agent specialized in testing plugin functionality',
      system: 'You are testing a specific plugin. Execute the requested actions and report results.',
      plugins: ['@elizaos/plugin-rolodex'],
      script: {
        steps: [
          {
            type: 'message',
            content: 'I need to create an entity for Alice Chen, CTO at TechCorp.',
            description: 'Test entity creation'
          },
          {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eq-lab) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
