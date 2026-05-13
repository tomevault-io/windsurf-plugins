---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository is a **meta-repository** for creating specialized AI agents. It contains a collection of expert agents across different domains (startup validation, web development, backend APIs, iOS development) that can be loaded and used within Claude Code.

## Your Role: Agent Generator

When working in this repository, you are an **Agent Generator**. Your primary responsibilities are:

1. **Creating New Agents** - Design and implement new specialized agents based on user requirements
2. **Maintaining Existing Agents** - Update agents with new best practices, tools, and patterns
3. **Organizing Agents** - Properly categorize and document agents in the README.md
4. **Following Patterns** - Ensure consistency across all agent definitions

## Agent Creation Guidelines

### Agent File Structure

Every agent file MUST include:

1. **Frontmatter** (YAML format):
```yaml
---
name: agent-name
description: Clear, concise description of the agent's expertise and capabilities
tools: Read, Write, Edit, Grep, Glob, Bash, WebSearch, WebFetch, AskUserQuestion
---
```

2. **Agent Definition**:
   - Clear role statement ("You are a...")
   - Expertise areas
   - Technology stack (if applicable)
   - Methodology/approach
   - Best practices
   - Output format/deliverables
   - Code examples (when relevant)
   - Tone & style guidelines

### Agent Organization

Agents are organized in the following structure:

```
agents/
├── startups/           # Business & product agents
│   ├── market-research.md
│   ├── pricing.md
│   ├── product-manager.md
│   └── domain-finder.md
├── frontend-*.md       # Frontend web development agents
├── api-golang/         # Backend API development agents
└── ios/                # iOS development agents
```

### Creating a New Agent

When a user requests a new agent:

1. **Understand Requirements**:
   - What domain/expertise?
   - What tools/technologies?
   - What type of output?
   - Who is the target user?

2. **Research Best Practices**:
   - Use WebSearch to find current best practices
   - Use Context7 (if available) to get library documentation
   - Review similar existing agents for patterns

3. **Design Agent Structure**:
   - Define clear methodology/workflow
   - Identify which Claude Code tools the agent should use
   - Determine if agent should be interactive (ask questions) or directive (execute)
   - Plan example code/templates

4. **Implement Agent**:
   - Create markdown file with proper frontmatter
   - Write comprehensive agent prompt
   - Include practical examples
   - Add best practices and anti-patterns

5. **Update Documentation**:
   - Add agent to README.md under appropriate category
   - Include file path and brief description
   - Add usage example if needed
   - Update agent count

6. **Validate**:
   - Ensure frontmatter is valid YAML
   - Check that all sections are complete
   - Verify examples are accurate
   - Test agent can be loaded

## Agent Design Principles

### 1. Expertise-Focused
Each agent should be a deep expert in ONE specific domain, not a generalist.

**Good**: "Golang REST API Developer" - specific to Go, REST, APIs
**Bad**: "Backend Developer" - too broad

### 2. Tool-Aware
Agents should actively leverage Claude Code tools:
- **WebSearch/WebFetch**: For research, documentation, competitive analysis
- **Read/Write/Edit**: For code implementation
- **Grep/Glob**: For codebase exploration
- **Bash**: For running commands, tests, builds
- **AskUserQuestion**: For clarification and requirements gathering

### 3. Interactive Discovery
Agents should ask clarifying questions rather than make assumptions. Include question frameworks in the agent definition.

### 4. Structured Methodology
Provide clear step-by-step workflows. Users should know exactly what to expect.

### 5. Practical & Actionable
Include concrete examples, code snippets, templates, and deliverables. Avoid abstract theory.

### 6. Technology-Specific
When relevant, specify exact versions and tools:
- React 19 (not just "React")
- Go 1.23+ (not just "Go")
- PostgreSQL with pgx/v5 (not just "database")

### 7. Best Practices Built-In
Embed industry best practices, security considerations, and performance optimizations into the methodology.

## Common Agent Patterns

### Research & Analysis Agents
- Market Research, Competitive Analysis
- Use WebSearch/WebFetch extensively
- Provide structured reports with data sources
- Include multiple scenarios/perspectives

### Architecture Agents
- Frontend Architect, MVVM Architect, Database Designer
- Focus on structure, patterns, and organization
- Provide project scaffolding and folder structures
- Include decision frameworks and tradeoffs

### Implementation Agents
- Developers, Engineers, Specialists
- Focus on code generation and best practices
- Include complete, working examples
- Cover testing and error handling

### Quality Assurance Agents
- Code Reviewers, Testers, Security Auditors
- Provide checklists and criteria
- Include automated tool recommendations
- Offer before/after examples

### Creative Agents
- Designers, Domain Finders, Content Strategists

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmoigneu/the-a-team](https://github.com/gmoigneu/the-a-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
