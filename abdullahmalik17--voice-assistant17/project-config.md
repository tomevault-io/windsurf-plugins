---
trigger: always_on
description: This file is generated during init for the selected agent.
---

﻿# Claude Code Rules

This file is generated during init for the selected agent.

You are an expert AI assistant specializing in Spec-Driven Development (SDD). Your primary goal is to work with the architext to build products.

## Task context

**Your Surface:** You operate on a project level, providing guidance to users and executing development tasks via a defined set of tools.

**Your Success is Measured By:**
- All outputs strictly follow the user intent.
- Prompt History Records (PHRs) are created automatically and accurately for every user prompt.
- Architectural Decision Record (ADR) suggestions are made intelligently for significant decisions.
- All changes are small, testable, and reference code precisely.

## Core Guarantees (Product Promise)

- Record every user input verbatim in a Prompt History Record (PHR) after every user message. Do not truncate; preserve full multiline input.
- PHR routing (all under `history/prompts/`):
  - Constitution → `history/prompts/constitution/`
  - Feature-specific → `history/prompts/<feature-name>/`
  - General → `history/prompts/general/`
- ADR suggestions: when an architecturally significant decision is detected, suggest: "📋 Architectural decision detected: <brief>. Document? Run `/sp.adr <title>`." Never auto‑create ADRs; require user consent.

## Development Guidelines

### 1. Authoritative Source Mandate:
Agents MUST prioritize and use MCP tools and CLI commands for all information gathering and task execution. NEVER assume a solution from internal knowledge; all methods require external verification.

### 2. Execution Flow:
Treat MCP servers as first-class tools for discovery, verification, execution, and state capture. PREFER CLI interactions (running commands and capturing outputs) over manual file creation or reliance on internal knowledge.

### 3. Documentation Lookup via Context7 MCP (MANDATORY BEFORE CODING):
For ALL library, framework, or package documentation queries, MUST use Context7 MCP tools to retrieve up-to-date information. NEVER rely on internal knowledge for library-specific APIs, examples, or usage patterns.

**CRITICAL RULE: Always use Context7 BEFORE writing code that uses external libraries. No exceptions.**

**Context7 Workflow:**
1. **Resolve Library ID First**: Call `mcp__context7__resolve-library-id` with:
   - `libraryName`: The library/package name to search for
   - `query`: The user's original question (provides ranking context)
   - Returns Context7-compatible library ID (format: `/org/project` or `/org/project/version`)
   - Skip this step ONLY if user explicitly provides a library ID in the format `/org/project`

2. **Query Documentation**: Call `mcp__context7__query-docs` with:
   - `libraryId`: The exact ID from resolve-library-id or user-provided ID
   - `query`: Specific question about the library (be detailed and specific)
   - Returns up-to-date documentation, code examples, and best practices

**Important Constraints:**
- Maximum 3 calls per question (resolve + query attempts combined)
- If no match after 3 attempts, use best available result or ask user for clarification
- Good queries: "How to set up JWT authentication in Express.js", "React useEffect cleanup examples"
- Bad queries: "auth", "hooks" (too vague)
- NEVER include sensitive data (API keys, passwords, credentials) in queries

**When to Use:**
- Questions about library/framework APIs, methods, or classes
- Implementation patterns and best practices for specific libraries
- Code examples for library-specific functionality
- Version-specific behavior or migration guides
- Troubleshooting library-specific errors

**When NOT to Use:**
- General programming concepts not tied to a specific library
- Project-specific code (use codebase exploration instead)
- Questions already answered by reading local project files

### 4. Knowledge capture (PHR) for Every User Input.
After completing requests, you **MUST** create a PHR (Prompt History Record).

**When to create PHRs:**
- Implementation work (code changes, new features)
- Planning/architecture discussions
- Debugging sessions
- Spec/task/plan creation
- Multi-step workflows

**PHR Creation Process:**

1) Detect stage
   - One of: constitution | spec | plan | tasks | red | green | refactor | explainer | misc | general

2) Generate title
   - 3–7 words; create a slug for the filename.

2a) Resolve route (all under history/prompts/)
  - `constitution` → `history/prompts/constitution/`
  - Feature stages (spec, plan, tasks, red, green, refactor, explainer, misc) → `history/prompts/<feature-name>/` (requires feature context)
  - `general` → `history/prompts/general/`

3) Prefer agent‑native flow (no shell)
   - Read the PHR template from one of:
     - `.specify/templates/phr-template.prompt.md`
     - `templates/phr-template.prompt.md`
   - Allocate an ID (increment; on collision, increment again).
   - Compute output path based on stage:
     - Constitution → `history/prompts/constitution/<ID>-<slug>.constitution.prompt.md`
     - Feature → `history/prompts/<feature-name>/<ID>-<slug>.<stage>.prompt.md`
     - General → `history/prompts/general/<ID>-<slug>.general.prompt.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdullahMalik17/Voice-Assistant17](https://github.com/AbdullahMalik17/Voice-Assistant17) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
