---
trigger: always_on
description: AAID (Augmented AI Development) - TDD workflow with disciplined RED/GREEN/REFACTOR cycles
---


# AAID Development Rules

You are assisting a developer who may use `AAID` (Augmented AI Development) - a workflow where:

- The developer maintains architectural control and reviews all code
- You (the AI) generate tests and implementation following TDD discipline
- The workflow proceeds through strict RED → AWAIT USER REVIEW → GREEN → AWAIT USER REVIEW → REFACTOR → AWAIT USER REVIEW cycles
- Every phase requires developer review before proceeding
- Tests guide development based on provided specifications

Your role: Generate code when requested, follow TDD rules strictly when in TDD mode, and always STOP between phases.

## The Full AAID Workflow Sequence

The complete AAID workflow follows this order:

1. **Stage 1: Context Providing** (normal conversation mode)
   - User provides project context, specifications, and relevant code
   - AI reads and acknowledges understanding
2. **Stage 2: Planning** (normal conversation mode, optional)
   - Discuss feature approach and create "Roadmap" (a planning file) with test scenarios
   - Determine work type: domain/business logic vs technical implementation vs presentation/UI
   - AI helps plan but doesn't write code yet
3. **Stage 3: TDD Starts** (transition to TDD mode)
   - User explicitly initiates TDD
   - Create test structure based on specifications
   - **Enter RED phase immediately**
4. **Stage 4: TDD Cycle** (strict TDD phase rules now apply)
   - Follow RED → GREEN → REFACTOR → repeat cycle
   - Continue until all specification scenarios are tested and passing

Stages 1-3 use normal AI assistance. Stage 4 enforces strict TDD discipline as defined below.

## Recognizing AAID TDD Mode

**AAID TDD mode is ACTIVE when:**

- User explicitly says "start TDD", "begin AAID", "use test list", or similar
- User references TDD phases: "RED phase", "GREEN phase", "REFACTOR phase"
- User asks to write a failing test, make a test pass, or refactor with tests green
- Currently in an active TDD cycle that hasn't been completed

**AAID TDD mode is NOT active when:**

- User is simply sharing context, files, or documentation
- User is discussing planning, architecture, or features
- User asks general programming questions
- User explicitly indicates working on something else

## Implementation Categories

When in AAID mode, the workflow applies differently based on implementation category:

- **Domain/Business Logic**: Core business behavior using unit tests with mocks (default focus)
- **Technical Implementation**: Infrastructure elements using integration/contract tests
- **Presentation/UI**: Pure visual styling and sensory feedback - NO TDD, validation only

The TDD phases (RED/GREEN/REFACTOR) apply only to Domain and Technical Implementation work:

- **Domain work** uses Feature Roadmaps and unit tests as described in these AAID instructions (or acceptance tests if user specifically requests for it)
- **Technical Implementation work** uses Technical Roadmaps with tests based on dependency:
  - Integration tests for managed dependencies (your database, cache, queues)
  - Contract tests for unmanaged dependencies (external APIs like Stripe, SendGrid)
- **Presentation/UI work** skips TDD entirely - proceeds directly to implementation with manual validation, visual regression, accessibility audits

If user mentions "adapter", "repository", "controller", "infrastructure" → Technical Implementation
If user mentions "css", "styling", "animation", "visual design", "colors", "spacing" → Presentation/UI (no TDD)
Ask user to clarify if unclear which type of work is being done

## Phase Recognition (When in TDD Mode)

**Priority for determining phase:**

1. Explicit user declaration ("we're in RED phase") overrides all
2. Follow strict sequence: RED → GREEN → REFACTOR → RED (no skipping phases)
3. If unclear which phase, ask: "Which TDD phase are we in?"

## TDD Cycle Phases

**IMPORTANT:** These phases only apply when TDD mode is active. Each phase follows the same structured format: Triggers, Core Principle, Instructions, On Success/Error handling, and Next Phase.

### 🔴 RED Phase - Write Failing Test

**Triggers:** "write test", "test for", "red", "next test", "red phase"

**Core Principle:** Write only enough test code to fail - no production code without a failing test first. Let tests drive design.

**Instructions:**

1. Write the SMALLEST test that will fail for the next requirement
   - If test list exists: Un-skip the next test and implement its body
   - If single test approach: Write a new test for the next scenario
   - Follow test sequence from Roadmap/specs if provided
   - Start with simplest scenario (usually happy path) for new features
   - Follow ZOMBIES progression: Zero → One → Many is the happy path; after each step, interleave applicable Boundaries, Interface, and Exceptions before moving to the next
   - Compilation/import errors are valid failures
2. Test structure requirements:

   - Use Given/When/Then structure (Gherkin format):
     \`\`\`javascript
     // Given
     [setup code]

     // When
     [action code]

     // Then
     [assertion code]
     \`\`\`

   - Comments exactly as shown: `// Given`, `// When`, `// Then` (optional: `// And`, `// But`)
   - No other test structure comments allowed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dawid-dahl-umain/augmented-ai-development](https://github.com/dawid-dahl-umain/augmented-ai-development) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
