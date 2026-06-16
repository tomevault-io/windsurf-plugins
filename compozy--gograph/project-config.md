---
trigger: always_on
description: Guide for creating Product Requirements Documents (PRDs) with AI assistance, including clarifying questions and structured documentation process
---

# Rule: Generating a Product Requirements Document (PRD)

<goal>
To guide an AI assistant in creating a detailed Product Requirements Document (PRD) in Markdown format, based on an initial user prompt. The PRD should be comprehensive, focusing on user needs, functional requirements, and business goals to clearly define *what* to build and *why*.
</goal>

## Template Reference

<template_reference>
**ALWAYS use the standardized PRD template:** tasks/docs/_prd-template.md

This template provides a comprehensive structure that balances requirements gathering with rollout planning, ensuring consistency across all PRDs in the project.
</template_reference>

## Process

<process_workflow>
1. **Receive Initial Prompt:** The user provides a brief description or request for a new feature or functionality.

2. **Ask Clarifying Questions:** Before writing the PRD, the AI *must* ask clarifying questions to gather sufficient detail. Focus on understanding the "what" and "why" of the feature, user needs, and success criteria.

**MANDATORY PLANNING STEPS:**
3. **Create PRD Planning with Zen Planner:** Use zen's planner tool to create a comprehensive PRD development plan:
   - Analyze the enriched feature specification requirements
   - Break down PRD creation into logical planning steps
   - Identify key sections that need focused attention
   - Plan resource allocation and approach for each section
   - Document assumptions and dependencies that will guide PRD creation

4. **Validate Planning with Consensus:** Use zen's consensus tool with o3 and gemini 2.5 models:
   - Present the detailed PRD planning approach to both expert models
   - Request critical analysis of the planning strategy
   - Gather feedback on plan completeness and effectiveness
   - Incorporate consensus recommendations into final planning approach
   - Proceed only after receiving aligned approval from both expert models

**PRD CREATION WORKFLOW:**
5. **Generate Comprehensive PRD (Functionality-Focused):** Using the template, produce a PRD that captures user and business requirements plus high-level product scope **without including low-level technical design or implementation details** – those belong in the Tech Spec.
6. **Create Feature Folder:** Instruct to create a feature folder `./tasks/prd-[feature-slug]/`.
7. **Save PRD:** Save the generated document as `_prd.md` inside the feature folder.
</process_workflow>

## Clarifying Questions (Examples)

<clarifying_questions_guidance>
The AI should adapt its questions based on the prompt and template sections. Here are key areas to explore:

**Problem & Goals:**
- "What problem does this feature solve for the user?"
- "What are the specific, measurable goals we want to achieve?"
- "How will we measure success?"

**Users & Stories:**
- "Who is the primary user of this feature?"
- "Can you provide user stories? (As a [type of user], I want to [action] so that [benefit])"
- "What are the key user flows and interactions?"

**Core Functionality:**
- "What are the essential features that must be included in the MVP?"
- "Can you describe the key actions a user should be able to perform?"
- "What data does this feature need to display or manipulate?"

**Technical Constraints (acceptance criteria only – describe *what* must be met, not *how* to meet it):**
- "Are there any existing systems this needs to integrate with?"
- "What are the performance thresholds or security requirements? (e.g., must handle X users, must comply with Y standard)"
- "Are there any technical constraints that limit what can be built?"
- Note: Capture only acceptance-criteria level thresholds; defer solution approaches to Tech Spec

**Scope & Planning:**
- "What should this feature NOT do (non-goals)?"
- "How should development be phased for incremental delivery?"
- "What are the dependencies between different parts of this feature?"

**Risks & Challenges:**
- "What are the biggest risks or challenges you foresee?"
- "Are there any unknowns that need research before implementation?"
- "What could prevent this feature from being successful?"

**Design & Experience:**
- "Are there any design mockups or UI guidelines to follow?"
- "What accessibility requirements should be considered?"
- "How should this feature integrate with the existing user experience?"
</clarifying_questions_guidance>

## PRD Structure Requirements

<prd_structure_requirements>
The generated PRD MUST follow the template structure from @_prd-template.md:

1. **Overview:** Problem statement, target users, and value proposition
2. **Goals:** Specific, measurable objectives and business outcomes
3. **User Stories:** Detailed narratives covering primary and edge case scenarios
4. **Core Features:** Main functionality with detailed functional requirements
5. **User Experience:** User journeys, flows, UI/UX considerations, and accessibility
6. **High-Level Technical Constraints:** Integration points, compliance mandates, performance thresholds (avoid architectural diagrams or code-level solutions)
7. **Non-Goals (Out of Scope):** Clear boundaries and excluded features
8. **Phased Rollout Plan:** User-facing milestones with MVP and enhancement stages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
