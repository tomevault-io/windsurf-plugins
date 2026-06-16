---
trigger: always_on
description: Guides AI in creating detailed Technical Specification documents that complement PRDs, providing senior-level architectural guidance and implementation details for development teams
---

# Rule: Generating Technical Specification Documents

<goal>
To guide an AI assistant in creating detailed Technical Specification (Tech Spec) documents that complement Product Requirements Documents (PRDs). The Tech Spec should provide senior-level architectural guidance, implementation details, and system design decisions suitable for implementation teams.
</goal>

## Template Reference

<template_reference>
**ALWAYS use the standardized Tech Spec template:** tasks/docs/_techspec-template.md
This template provides a comprehensive structure for technical specifications, ensuring consistency across all technical documentation in the project.
</template_reference>

## Prerequisites

<prerequisites>
- **Review Project Standards:** First examine `.cursor/rules/` to understand all established coding standards, patterns, and architectural guidelines
- **MANDATORY:** Review [architecture.mdc](mdc:.cursor/rules/architecture.mdc) for SOLID principles, Clean Architecture, and design patterns
- A completed PRD document must exist in `tasks/prd-[feature-slug]/_prd.md`
- Understanding of the project's domain structure: `engine/{agent,task,tool,workflow,runtime,infra,llm,mcp,project,schema,autoload,core}/` and `pkg/` for internal packages
- Familiarity with project standards: Go patterns, clean architecture, SOLID principles
- **Separation of Concerns:** Confirm that detailed technical design and implementation information is **absent** from the PRD and will be authored exclusively in this Tech Spec. If such details are found in the PRD, create a `PRD-cleanup.md` file in the feature folder listing the specific lines to remove or move, and notify the PRD owner.
</prerequisites>

## Process

<process_workflow>
1. **Analyze PRD:** Review the existing PRD to understand requirements and scope, noting any misplaced technical details that should be migrated to the Tech Spec.
2. **Pre-Analysis with Zen MCP:** Use Zen MCP with Gemini 2.5 and O3 to analyze the PRD and identify technical complexity areas, potential architecture patterns, and system design considerations
3. **Ask Technical Questions:** Gather technical clarifications focusing on system design, performance, and architecture decisions
4. **Generate Tech Spec:** Create focused technical specification appropriate for MVP/alpha phase
5. **Post-Review with Zen MCP:** Use Zen MCP with Gemini 2.5 and O3 to review the generated tech spec for completeness, architectural soundness, and adherence to best practices
6. **Save Tech Spec:** Save as `_techspec.md` in `tasks/prd-[feature-slug]/`
</process_workflow>

## Technical Clarifying Questions

<technical_questions_guidance>
Focus on essential implementation details for MVP:

* **System Architecture:** "Which domain should this feature belong to? (agent/task/tool/workflow/runtime/infra/llm/mcp/project/schema/autoload/core or pkg/ for internal packages)"
* **Data Flow:** "What are the main data inputs/outputs and how do they flow through the system?"
* **External Dependencies:** "Does this require any external services or APIs?"
* **Key Implementation:** "What's the core logic that needs to be implemented?"
* **Testing Focus:** "What are the critical paths that must be tested?"
* **Impact Analysis:** "What existing modules or components might be affected by this change?"
* **Monitoring:** "What metrics or logs would be useful for monitoring this feature?"
* **Special Concerns:** "Are there any specific performance or security requirements for this feature?"

Note: Keep questions focused on what's needed for implementation, not theoretical concerns.
</technical_questions_guidance>

## Tech Spec Structure

<spec_structure>
The generated Tech Spec MUST follow the template structure from @_techspec-template.md:

1. **Executive Summary:** Brief technical overview (1-2 paragraphs)
2. **System Architecture:** Domain placement and component overview
3. **Implementation Design:** Core interfaces, data models, and API endpoints
4. **Integration Points:** External integrations (only if needed)
5. **Impact Analysis:** Effects on existing components, APIs, and data models
6. **Testing Approach:** Unit and integration testing strategy
7. **Development Sequencing:** Build order and technical dependencies
8. **Monitoring & Observability:** Metrics, logs, and dashboards
9. **Technical Considerations:** Key decisions, risks, special requirements, and standards compliance

Note: Keep the spec focused and practical. Only include sections relevant to the feature. The goal is a 2-4 page document that developers can quickly reference.
</spec_structure>

## Content & Code Snippet Guidelines

<content_guidelines>
**Primary Focus:** Provide architectural decisions, system design rationale, and interface-level detail. Avoid implementation-heavy content.

**Code Snippets:**
- Include **only** concise, illustrative code snippets (≤ 20 lines).
- Focus on interfaces, method signatures, or small algorithm excerpts necessary to explain a concept.
- **NEVER** embed complete source files or large implementation blocks.
- Use `// ...` or `/* ... */` to indicate omitted sections when context is needed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/gograph](https://github.com/compozy/gograph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
