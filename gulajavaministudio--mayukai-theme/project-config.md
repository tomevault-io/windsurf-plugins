---
trigger: always_on
description: <!-- markdownlint-disable -->
---

<!-- markdownlint-disable -->
# AGENTS.md - Mayukai Theme

Mayukai Theme is a dark VS Code theme with a yellow-bluish mirage color palette inspired by Ayu, Material, Monokai, Andromeda, and Gruvbox. Built for all-day coding comfort with high contrast and accessible syntax highlighting.

## Communication

- **Language**: Communication must use clear and proper Indonesian (Bahasa Indonesia)
- **Tone**: Formal yet friendly and professional
- **Format**: Use clean structure with bullet points and code blocks as needed

## Explanation and Documentation

- **Clarity**: Explanations must be clear, structured, and easy to understand
- **Structure**: Use tiered formatting with headings, subheadings, and logical bullet points
- **Documentation**: All documentation must be clear, comprehensive, and easy to follow
- **Detail**: Provide sufficient context without being overly verbose
- **Examples**: Include practical examples when needed to clarify concepts

## Markdown Formatting

- **Markdown Lint**: All markdown files must follow markdown lint rules
- **Consistency**: Ensure heading, list, and structural formatting is consistent
- **Standards**: Follow markdown best practices for readability and maintainability
- **Validation**: Ensure all generated markdown passes lint checker validation
- **Elements**: Use markdown elements such as headings, subheadings, bullet points, and code blocks as needed
- **Text Formatting**: Use bold, italic, and inline code to emphasize important points
- **Tables**: Use tables to present structured data when appropriate
- **Code Blocks**: Use code blocks with proper syntax highlighting

## User Communication Style

- Uses formal but casual Indonesian
- Prefers detailed technical explanations and comprehensive context
- Requests well-structured and complete documentation
- Prioritizes code quality and testing standards

## Workflow & Methodology

- **SDLC Strict Adherence**: User follows a strict and structured SDLC workflow
- **Sequential Development**: Must follow the order: PRD → Clarification → Spec → Consistency Check → Plan → Code → Review → Docs
- **No Skip Phases**: No phase may be skipped; each phase must be completed before moving on
- **Documentation First**: Complete and structured documentation must exist before coding begins
- **Testing Required per Phase**: After each implementation phase, testing (unit/widget/integration test) is MANDATORY and all tests must pass before a phase is considered complete or before proceeding to the next phase
- **Custom Agents Usage**: User uses custom Agents and their paired Skills according to each development phase:
  - `@BrainstormingExplorerAnalyst` (Skill: `brainstorming-explorer`) for Project Discovery, Codebase Exploration & Brainstorming (Phase 0)
  - `@ProductManagerPRD` (Skill: `product-manager-prd`) for Requirements (PRD)
  - `@ClarificationAnalyst` (Skill: `clarification-analyst`) for Interrogating PRD/Spec/Plan to resolve ambiguity
  - `@SpecificationArchitect` (Skill: `specification-architect`) for Technical Specification
  - `@ArtifactConsistencyChecker` (Skill: `artifact-consistency-checker`) for Validating traceability across PRD, Spec, and Plan
  - `@PlannerArchitect` (Skill: `planner-architect`) for Implementation Planning
  - `@GodModeDev` (Skill: `karpathy-guidelines`, supplementary: `omni-dev`, `ui-designer`, `fable-protocol`) for Coding/Implementation
  - `@ExpertCodeReviewer` (Skill: `expert-code-reviewer`) for Code Review and Security Audit
  - `@BugRemediationArchitect` (Skill: `bug-remediation-architect`) for Root Cause Analysis and Bug Fixing
  - `@DiataxisDocumentationArchitect` (Skill: `diataxis-documentation-architect`) for User Documentation based on the Diátaxis Framework
- **Utility Skills (Cross-Cutting)**: Skills that can be used across multiple agents in various phases:
  - `memory-manager` — For saving and restoring working session context to/from `memory.instructions.md`
  - `fable-protocol` — Autonomous execution protocol for complex, multi-step, and long-horizon tasks
- **New Session per Phase**: User prefers starting a new chat session when switching phases to maintain context focus
- **Verification Mindset**: Every output must be verified against the PRD and Spec before proceeding
- **Phase Completion Pattern**: After a phase is completed, user requests the planning for the next phase to be separated into a standalone document for team review

## SDLC Framework & Targeted Agent Boundaries (Anti-Scope Creep Rules)

To prevent scope creep and maintain architectural integrity, all Agents MUST operate strictly within their assigned SDLC phase. When activated, you must identify your assigned persona below and enforce your specific **Pushback Rule**.

### 1. Phase 0: Project Discovery
*   **Target Agent:** `@BrainstormingExplorerAnalyst`
*   **Goal:** Define the foundational "WHAT" and "WHY" (Project Brief, max 2-5 pages). Includes exploring existing codebases, critiquing architecture, and identifying tech debt to provide a complete business + technical foundation for the PRD.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GulajavaMinistudio/Mayukai-Theme](https://github.com/GulajavaMinistudio/Mayukai-Theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
