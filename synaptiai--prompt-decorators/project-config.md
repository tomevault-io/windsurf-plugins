---
trigger: always_on
description: As an AI language model, you must identify and process "Prompt Decorators" in user queries. A Prompt Decorator is an annotation that begins with `+++` followed by a name and optional parameters that modifies how prompts are generated and structured. When you detect these decorators in a user's input, apply the corresponding transformation instructions to enhance your response.
---

# Prompt Decorators Processing Instructions

## Overview
As an AI language model, you must identify and process "Prompt Decorators" in user queries. A Prompt Decorator is an annotation that begins with `+++` followed by a name and optional parameters that modifies how prompts are generated and structured. When you detect these decorators in a user's input, apply the corresponding transformation instructions to enhance your response.

This document defines all supported decorators, their parameters, and how to transform the prompt when they are detected. You must parse and apply these rules whenever you encounter decorators in user input.

## Decorator Detection and Application Process
1. When receiving a user prompt, scan for patterns matching `+++DecoratorName(parameters)` at the beginning of lines
2. For each detected decorator, apply its transformation instruction to modify your processing approach
3. When multiple decorators are present, apply them in the order they appear
4. Maintain the core query intent while applying the decorator-specific modifications to your response style, structure, and content

---

## Core Prompt Decorators

### `+++Reasoning`
When this decorator is included in a prompt, the response must provide explicit reasoning paths before reaching conclusions. This makes the thinking process transparent.

**Parameters:**
- `depth (basic | moderate | comprehensive)`: Controls the detail level of reasoning
  - `basic`: Focus on the most important logical steps
  - `moderate`: Balance detail with clarity
  - `comprehensive`: Provide thorough analysis with multiple perspectives

**Transformation Instruction:**
"Please provide detailed reasoning in your response. Show your thought process before reaching a conclusion. [depth-specific instruction]"

**Example:**
```
+++Reasoning(depth=comprehensive)
What are the implications of quantum computing for cybersecurity?
```

---

### `+++StepByStep`
When this decorator is present, the response must be structured as a sequence of clearly labeled steps, making complex processes more digestible.

**Parameters:**
- `numbered (true | false)`: Whether to number steps sequentially or use bullet points

**Transformation Instruction:**
"Please break down your response into clear, sequential steps. [If numbered=true: Number each step sequentially (Step 1, Step 2, etc.). If numbered=false: Use bullet points for each step instead of numbers.]"

**Example:**
```
+++StepByStep(numbered=true)
How do I set up a Python virtual environment?
```

---

### `+++OutputFormat`
When this decorator is applied, the response must adhere to the specified output format structure.

**Parameters:**
- `format (json | markdown | yaml | xml | plaintext)`: Specifies the desired output format

**Transformation Instruction:**
"Format your entire response as [format]. Ensure the output is valid and properly structured."

**Example:**
```
+++OutputFormat(format=json)
List the top 5 programming languages and their key features.
```

---

### `+++Tone`
When this decorator is used, the response tone must match the specified style throughout.

**Parameters:**
- `style (formal | casual | friendly | technical | humorous)`: Defines the tone of the response

**Transformation Instruction:**
"Please respond in a [style] tone throughout your answer."

**Example:**
```
+++Tone(style=technical)
Explain how blockchain works.
```

---

### `+++Layered`
When this decorator is applied, the response is structured in multiple conceptual layers with increasing depth and complexity.

**Parameters:**
- `levels (ONE | TWO | THREE | FIVE)`: Controls the number of layers
- `progression (LINEAR | EXPONENTIAL | FRACTAL)`: Defines how layers build upon each other

**Transformation Instruction:**
"Structure your response in [levels] distinct layers of understanding, with each layer [progression type] in complexity from the previous one. Begin with the most accessible explanation and progressively add depth."

**Example:**
```
+++Layered(levels=THREE, progression=LINEAR)
Explain how neural networks function.
```

---

### `+++ForcedAnalogy`
When this decorator is included, the response must incorporate relevant analogies to explain concepts.

**Parameters:**
- `comprehensiveness (BASIC | INTERMEDIATE | COMPREHENSIVE | EXHAUSTIVE)`: Controls how detailed the analogies are

**Transformation Instruction:**
"Please incorporate [comprehensiveness level] analogies in your explanation to make the concepts more accessible. Use familiar comparisons to illustrate key points."

**Example:**
```
+++ForcedAnalogy(comprehensiveness=COMPREHENSIVE)
Explain how the internet works.
```

---

### `+++Deductive`
When this decorator is applied, the response follows formal deductive reasoning patterns, moving from general principles to specific conclusions.

**Parameters:**
- `premises (1-5)`: Number of main premises to include
- `formal (true | false)`: Whether to use formal logical structures

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [synaptiai/prompt-decorators](https://github.com/synaptiai/prompt-decorators) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
