---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AgentScope 2.0 is a production-ready multi-agent framework for building LLM-empowered agent applications. This repository contains the documentation for the AgentScope framework, built with Mintlify. 

The goal of this documentation is to help developers understand "how to use" the AgentScope 2.0 framework. Explaining "how AgentScope works or implements a feature internally" is secondary — your goal is to enable readers to use each part of the framework based on the documentation.

## Writing Style Guidelines

Follow these core principles when writing documentation:

### The Source Code Helps
- AgentScope is open-sourced in https://github.com/agentscope-ai/agentscope
- Read the source code helps you better understand and translation

### Language and Style
- Use clear, concise, precise, and direct language appropriate for technical audiences
- Address the reader as "developer" in instructions and procedures (use "开发者" in Chinese documentation)
- Use active voice over passive voice
- Maintain consistent terminology throughout all documentation
- Minimize em dashes (`—` in English, `——` in Chinese) in both languages. Prefer commas, colons, parentheses, or splitting into two sentences. When a bullet list pairs each item with a dash-joined explanation (e.g. "单个 `Msg` 或 `Msg` 列表——开始一次新的回复"), turn the list into a table instead. Em dashes as empty-cell placeholders in tables are fine

### Content Organization
- Lead with the most important information (inverted pyramid structure)
- Use progressive disclosure: basic concepts before advanced ones
- Break complex procedures into numbered steps
- Include prerequisites and context before instructions
- Provide expected outcomes for each major step
- Always precede tables, images, diagrams, and code blocks with an explanatory sentence or paragraph that describes what the content shows or demonstrates
- Avoid long, dense paragraphs — they make documentation hard to read and understand; break content down with code, bullet points, steps, tables, etc.

A representative page structure:

1. The first paragraph introduces the feature very concisely (usually one sentence): what it does — so readers know what this section covers. Do not use descriptions like "This section introduces ..."; instead, introduce the feature directly, e.g. "When xxx (scenario), you can achieve xxx by setting the xxx field of xxx"
Note: do not copy this example sentence verbatim — in the English version, use more natural, idiomatic English phrasing.
2. The second paragraph introduces the concepts/features, also serving as an overview and lead-in; it can incorporate tables, diagrams, code, etc.
3. Introduce each part in separate sections
  - For important interfaces, explain how to use them in detail
  - For abstractions, introduce the implementations currently supported in the repo

## Mintlify Component Usage

### Callout Components
- `<Note>` - Supplementary information that supports the main content
- `<Tip>` - Expert advice or best practices
- `<Warning>` - Critical information about potential issues
- `<Info>` - Background information or context
- `<Check>` - Success confirmations or positive indicators

### Code Components
- **Single code block**: Include language specification when possible
  ```javascript config.js
  const apiConfig = {
    baseURL: 'https://api.example.com',
    timeout: 5000
  };
  ```

- **Code groups**: Use `<CodeGroup>` for multiple implementation options
  ````
  <CodeGroup>
  ```javascript Node.js
  const response = await fetch('/api/endpoint');
  ```

  ```python Python
  import requests
  response = requests.get('/api/endpoint')
  ```
  </CodeGroup>
  ````

- **API examples**: Use `<RequestExample>` and `<ResponseExample>` for API documentation

### Structural Components
- **Steps**: Use `<Steps>` for sequential instructions
- **Tabs**: Use `<Tabs>` for platform-specific content
- **Accordions**: Use `<AccordionGroup>` for expandable content
- **Cards**: Use `<Card>` and `<CardGroup>` to highlight important information
  - The `<Card>` component must have `title`, `icon`, `href`, and `cta` fields

### API Documentation
- **Parameters**: Use `<ParamField>` to document API parameters
- **Responses**: Use `<ResponseField>` to document API response properties
- **Nested objects**: Use `<Expandable>` for hierarchical information

## Page Structure

Every documentation page must begin with YAML frontmatter:
```yaml
---
title: "Clear, specific, keyword-rich title"
description: "Concise description explaining page purpose and value"
---
```

The `description` must read like one natural sentence, not a keyword list or a colon/dash-joined feature enumeration. Good examples (in the spirit of agno's docs, do not copy them verbatim): "Run agents and process their output.", "Start simple: a model, tools, and instructions.", "Groups of agents that collaborate to solve complex tasks.". Bad example: "Drive the reasoning-acting loop: reply, stream, structured output, observation, compression, and persistence".


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentscope-ai/docs](https://github.com/agentscope-ai/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
