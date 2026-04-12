---
trigger: always_on
description: description: WHEN starting a new session EXECUTE initialization procedure to establish memory context
---

---
description: WHEN starting a new session EXECUTE initialization procedure to establish memory context
globs: ["**/*.md", "**/*.mdc"]
alwaysApply: true
---

> **TL;DR:** The Standard Initialization Procedure provides a systematic process for memory bank initialization at the beginning of each session, ensuring complete context restoration despite memory reset.

<version>1.0.0</version>

<context>
  This rule establishes the Standard Initialization Procedure that must be executed at the start of every session. Since BIG BRAIN's memory resets completely between sessions, a standardized initialization process is critical for ensuring all necessary context is properly loaded and verified. This procedure ensures that operations begin with a complete understanding of the project state and a fully functional memory bank.
</context>

<requirements>
  <requirement>Execute at the beginning of every session without exception</requirement>
  <requirement>Systematically restore full context from memory bank</requirement>
  <requirement>Verify integrity and consistency of all memory files</requirement>
  <requirement>Establish appropriate task complexity framework</requirement>
  <requirement>Confirm operational readiness for the current session</requirement>
  <requirement>Adapt initialization depth to task requirements</requirement>
</requirements>

<details>
  <section-name>INITIALIZATION PROCESS</section-name>
  <content>
    The initialization procedure follows this sequence:

    1. **Identity Confirmation**
       - Establish BIG BRAIN identity
       - Acknowledge memory reset
       - Recognize memory bank dependency
       - Set operational parameters

    2. **Memory Bank Loading**
       - Locate core memory files
       - Load essential context documents
       - Verify file availability
       - Establish memory structure

    3. **Context Restoration**
       - Process project brief
       - Understand product context
       - Absorb active context
       - Comprehend system patterns
       - Assimilate technical context
       - Review progress status

    4. **Verification Execution**
       - Perform file verification
       - Execute consistency checks
       - Validate references
       - Assess memory health
       - Generate verification report

    5. **Task Contextualization**
       - Determine current focus
       - Assess task complexity
       - Select appropriate workflow
       - Establish operational mode
       - Identify relevant patterns

    6. **Readiness Confirmation**
       - Confirm complete context loading
       - Verify memory bank health
       - Establish system capabilities
       - Note any limitations
       - Declare initialization complete

    This sequence ensures systematic restoration of complete operational context.
  </content>
</details>

<details>
  <section-name>CORE MEMORY FILES</section-name>
  <content>
    These critical files must be loaded during initialization:

    1. **projectbrief.md**
       - Foundation document
       - Core requirements
       - Project goals and scope
       - Success criteria

    2. **productContext.md**
       - Problem space
       - User experience goals
       - Value proposition
       - Market context

    3. **activeContext.md**
       - Current work focus
       - Recent changes
       - Next steps
       - Open issues

    4. **systemPatterns.md**
       - System architecture
       - Component relationships
       - Implementation patterns
       - Design decisions

    5. **techContext.md**
       - Technology stack
       - Development environment
       - Technical constraints
       - Integration points

    6. **progress.md**
       - Implementation status
       - Feature completeness
       - Known issues
       - Future milestones

    These files provide the essential context required for operation.
  </content>
</details>

<details>
  <section-name>INITIALIZATION COMMAND</section-name>
  <content>
    Initialization is triggered through the command:

    ```
    BIG initialize [task] [--complexity=1-4] [--mode=plan|act] [--focus=area]
    ```

    Parameters:
    - **task**: Brief description of the current task
    - **complexity**: Task complexity level (1-4)
    - **mode**: Primary operation mode (plan or act)
    - **focus**: Specific area of focus

    Examples:
    ```
    BIG initialize "Implement authentication system" --complexity=3 --mode=plan
    BIG initialize "Fix login bug" --complexity=1 --mode=act --focus=auth
    BIG initialize "Design database schema" --complexity=2
    ```

    The command triggers the full initialization sequence with appropriate parameters.
  </content>
</details>

<details>
  <section-name>VERIFICATION REQUIREMENTS</section-name>
  <content>
    Initialization verification includes:

    1. **File Completeness**
       - All required memory files present
       - Core sections available in each file
       - Critical information accessible
       - Reference structure intact

    2. **Content Integrity**
       - No corruption in memory files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/savagelysubtle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
