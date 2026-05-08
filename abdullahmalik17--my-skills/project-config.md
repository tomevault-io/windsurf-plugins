---
trigger: always_on
description: Use when users need to create new documents or work with tracked changes.
---

# My Skills - Reusable Claude Code Skills Library

## Overview

This repository contains a curated collection of 50+ reusable skills designed to extend Claude Code's capabilities across various software engineering domains. Each skill encodes battle-tested patterns, domain expertise, and procedural knowledge that transforms Claude from a general-purpose agent into a specialized expert.

## Repository Structure

```
My_skills/
├── CLAUDE.md                    # This file - project documentation
├── README.md                    # Project overview
└── .claude/
    └── skills/                  # Skills library
        ├── SKILLS-INDEX.md      # Comprehensive skills catalog
        └── [skill-name]/        # Individual skill packages
            ├── SKILL.md         # Skill definition and instructions
            ├── scripts/         # Executable automation scripts
            ├── references/      # Deep-dive documentation
            └── assets/          # Output templates and resources
```

## What Are Skills?

Skills are **NOT documentation**. They are self-contained packages that encode:

- **Frozen decisions**: Judgment calls that took hours or days to determine
- **Production gotchas**: Edge cases and failure modes discovered in real deployments
- **Expert patterns**: Workflows that domain experts use but rarely document
- **Tool combinations**: Proven integrations between multiple technologies

**The test:** If you would need to recreate this knowledge after deletion, and Claude doesn't already know it, then it belongs in a skill.

## Skill Categories

This repository organizes skills into eight categories:

### 1. MCP-Backed Skills (3)
Require MCP (Model Context Protocol) servers for functionality.
- `browsing-with-playwright` - Browser automation
- `fetching-library-docs` - Library documentation retrieval
- `researching-with-deepwiki` - Repository research

### 2. Infrastructure (3)
Container orchestration and Kubernetes workflows.
- `containerizing-applications` - Docker and Helm patterns
- `operating-k8s-local` - Local Kubernetes development
- `deploying-cloud-k8s` - Cloud deployment pipelines

### 3. Application Development (4)
Full-stack application patterns and frameworks.
- `building-nextjs-apps` - Next.js 16+ patterns
- `scaffolding-fastapi-dapr` - Microservices architecture
- `configuring-better-auth` - Authentication systems
- `building-rag-systems` - Retrieval-augmented generation

### 4. UI/Frontend (4)
Chat interfaces and component styling.
- `styling-with-shadcn` - Component library integration
- `building-chat-interfaces` - AI chat UI patterns
- `building-chat-widgets` - Embeddable chat widgets
- `streaming-llm-responses` - Real-time streaming

### 5. Development Practices (2)
Debugging and production operations.
- `systematic-debugging` - Four-phase debugging methodology
- `operating-production-services` - SLO alerting and incident response

### 6. Document Processing (2)
Office file manipulation and generation.
- `working-with-spreadsheets` - Excel and financial models
- `working-with-documents` - Word, PDF, PowerPoint

### 7. Meta Skills (2)
Skills for creating and managing skills.
- `creating-skills` - Skill creation framework
- `installing-skill-tracker` - Usage analytics

### 8. Integration (2)
External service and tool integration patterns.
- `building-mcp-servers` - MCP server development
- `internal-comms` - Corporate communication templates

## Working with This Repository

### For Claude: Skill Usage Guidelines

When working with this repository:

1. **Skill Selection**
   - Check the `description` field in SKILL.md frontmatter for trigger conditions
   - Match user intent, not just keywords
   - Verify exclusions (some skills specify "NOT when...")
   - Choose the most specific skill when multiple could apply

2. **Progressive Loading**
   - Skill metadata (name + description) is always loaded (~100 tokens)
   - SKILL.md body loads when skill triggers (<5000 tokens)
   - References and scripts load as needed during execution

3. **Skill Dependencies**
   - Some skills work together (see SKILLS-INDEX.md for dependencies)
   - Follow cross-references in "Related Skills" sections
   - Chain skills when tasks span multiple domains

4. **Verification**
   - Each skill includes `scripts/verify.py` for validation
   - Run verification before packaging or distribution

### For Developers: Contributing Skills

#### Before Creating a Skill

Ask: **Does Claude actually need this?**

- Standard library usage → NO
- Well-documented public patterns → NO
- Spent hours debugging it → YES
- Broke production because of it → YES
- Domain expert knowledge → YES

#### Skill Creation Workflow

1. **Understand with examples**
   - Gather concrete use cases
   - Identify trigger phrases
   - Document expected inputs/outputs

2. **Plan reusable contents**
   - Identify repetitive code → scripts/
   - Identify reference docs → references/
   - Identify output templates → assets/

3. **Initialize skill**
   ```bash
   python3 .claude/skills/creating-skills/scripts/init_skill.py <skill-name>
   ```

4. **Implement skill**
   - Write YAML frontmatter (name, description)
   - Author markdown instructions
   - Create bundled resources
   - Keep SKILL.md under 500 lines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdullahMalik17/My_skills](https://github.com/AbdullahMalik17/My_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
