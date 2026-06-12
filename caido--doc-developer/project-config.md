---
trigger: always_on
description: This repository contains the **Caido Developer Documentation** - a comprehensive documentation site for developers building plugins for Caido, a lightweight web security auditing toolkit.
---

# AGENTS.md - AI Agent Contribution Guide

## Repository Overview

This repository contains the **Caido Developer Documentation** - a comprehensive documentation site for developers building plugins for Caido, a lightweight web security auditing toolkit.

The documentation is built using [VitePress](https://vitepress.dev/) and is published at [docs.caido.io](https://docs.caido.io). All documentation is written in Markdown format.

## Documentation Framework: Diátaxis

This repository **strictly adheres** to the [Diátaxis documentation framework](https://diataxis.fr/). Diátaxis organizes documentation into four distinct types based on user needs:

### The Four Documentation Types

1. **Tutorials** (`src/tutorials/`)
   - **Purpose**: Learning-oriented, step-by-step lessons
   - **User need**: "I want to learn how to do something"
   - **Characteristics**:
     - Shows how to accomplish a specific task from start to finish
     - Provides a working example that can be followed
     - Teaches through doing
     - Should result in a working implementation
   - **Example**: The Notebook tutorial walks through building a complete note-taking plugin

2. **How-to Guides** (`src/guides/`)
   - **Purpose**: Task-oriented, problem-solving instructions
   - **User need**: "I want to know how to accomplish a specific task"
   - **Characteristics**:
     - Focuses on solving a particular problem or achieving a specific goal
     - Provides clear, actionable steps
     - Assumes some prior knowledge
     - Answers "how do I...?" questions
   - **Example**: "How to create a command", "How to query requests"

3. **Reference** (`src/reference/`)
   - **Purpose**: Technical documentation, API specifications
   - **User need**: "I need to look up how something works"
   - **Characteristics**:
     - Describes the machinery - how things work
     - Accurate, complete, and concise
     - Organized for quick lookup
     - No explanation of why, just what
   - **Example**: SDK API documentation, manifest.json field definitions, module references

4. **Explanation** (`src/concepts/`)
   - **Purpose**: Understanding-oriented, conceptual documentation
   - **User need**: "I want to understand how something works"
   - **Characteristics**:
     - Explains background, context, and reasoning
     - Helps users understand the "why" behind things
     - Provides conceptual understanding
     - May include diagrams, metaphors, and examples
   - **Example**: "Plugin Architecture", "Dealing with Binary Data", "Plugins vs Workflows"

### Key Principles

- **Each type serves a different need** - Don't mix purposes. A tutorial should not become a reference guide.
- **Users know what they need** - Structure documentation so users can quickly find the right type.
- **Clear boundaries** - Keep content within its appropriate category.

## Repository Structure

```text
src/
├── tutorials/          # Step-by-step learning experiences
│   ├── index.md
│   └── notebook.md
├── guides/             # Task-oriented how-to guides (all files flattened)
│   ├── index.md
│   ├── config.md
│   ├── vibe_coding.md
│   ├── page.md
│   ├── command.md
│   ├── menu.md
│   ├── styling.md
│   ├── request.md
│   ├── fetch.md
│   ├── querying_requests.md
│   ├── utf.md
│   ├── findings.md
│   ├── env.md
│   ├── frontend_storage.md
│   ├── sqlite.md
│   ├── files.md
│   ├── rpc.md
│   ├── backend_events.md
│   ├── events.md
│   ├── spawning_process.md
│   ├── repository.md
│   ├── store.md
│   └── documentation.md
├── reference/          # Technical reference documentation
│   ├── index.md
│   ├── sdks/          # Generated SDK documentation
│   ├── modules/       # Generated module documentation
│   ├── config.md
│   ├── manifest.md
│   ├── plugin_packages.md
│   ├── api.md
│   └── authentication.md
└── concepts/          # Explanatory documentation (all files flattened)
    ├── index.md
    ├── package.md
    ├── tooling.md
    ├── runtime.md
    ├── signing.md
    ├── ui.md
    ├── binary.md
    ├── workflow.md
    └── child_process.md
```

## How to Contribute (For AI Agents)

### 1. Determine the Correct Documentation Type

Before writing or editing documentation, determine which type it should be:

- **Tutorial**: Is this teaching someone how to build something from scratch? → `tutorials/`
- **How-to Guide**: Is this solving a specific problem or task? → `guides/`
- **Reference**: Is this documenting an API, function, or configuration? → `reference/`
- **Explanation**: Is this explaining concepts, architecture, or reasoning? → `concepts/`

### 2. File Organization

- All documentation files are Markdown (`.md`)
- Files must be linked in the appropriate `index.md` file to appear in navigation
- Use descriptive, lowercase filenames with hyphens (e.g., `creating-commands.md`)
- **Guides are now flattened**: All guide files are directly in `src/guides/` (no subdirectories)
- **Concepts are now flattened**: All concept files are directly in `src/concepts/` (no subdirectories)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caido/doc-developer](https://github.com/caido/doc-developer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
