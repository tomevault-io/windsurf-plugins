---
trigger: always_on
description: ALWAYS INCLUDE to HAVE Project Context
---

---
description: ALWAYS INCLUDE to HAVE Project Context.
globs: 
alwaysApply: true
---
# Memory Files Structure
This outlines the fundamental principles, required files, workflow structure, and essential procedures that govern documentation, and maintaining a memory using file system.
The Memory Files consists of required core files and optional context files. Files build upon each other in a clear hierarchy:
```mermaid
flowchart TD
    PB[ [product_requirement_docs.md](mdc:docs/product_requirement_docs.md) ] --> PC[ [technical.md](mdc:docs/technical.md) ]
    PB --> SP[ [architecture.md](mdc:docs/architecture.md) ]

    SP --> TC[ [tasks_plan.md](mdc:tasks/tasks_plan.md) ]
    PC --> TC
    PB --> TC
    
    TC --> AC[ [active_context.md](mdc:tasks/active_context.md) ]

    AC --> ER[ [error-documentation.mdc](mdc:.cursor/rules/error-documentation.mdc)]
    AC --> LL[ [lessons-learned.mdc](mdc:.cursor/rules/lessons-learned.mdc) ]
    
    subgraph LIT[ @docs/literature ]
        L1[...]
        L2[...]
    end
    
    subgraph RFC[ @tasks/rfc/ ]
        R1[...]
        R2[...]
    end
    
    PC --o LIT
    TC --o RFC

```
## Core Files (Required)
  7 files: 
  1. [product_requirement_docs.md](mdc:docs/product_requirement_docs.md) (docs/product_requirement_docs.md): Product Requirement Document (PRD) for the project or an SOP. 
  - Why this project exists
  - Problems it solves
  - Defines core requirements and goals
  - Foundation document that shapes all other files
  - Source of truth for project scope
  - Created at project start if it doesn't exist

  2. [architecture.md](mdc:docs/architecture.md) (docs/architecture.md): System architecture
  - How it should work
  - Component relationships
  - Dependencies
  - Work flow of the Solution
  - Mermaid diagram of solution overview: each component and code flow

  3. [technical.md](mdc:docs/technical.md) (docs/technical.md): Development environment and stack
  - Technologies used
  - Development setup
  - Key technical decisions
  - Design patterns in use
  - Technical constraints

  4. [tasks_plan.md](mdc:tasks/tasks_plan.md) (tasks/tasks_plan.md): Detailed Task backlog
  - In-Depth Tasks list and Project Progress
  - What works
  - What's left to build
  - Current status
  - Known issues
  
  5. [active_context.md](mdc:tasks/active_context.md) (tasks/active_context.md): Current state of development
  - Current work focus
  - Active decisions and considerations
  - Recent changes
  - Next steps

  6. [error-documentation.mdc](mdc:.cursor/rules/error-documentation.mdc) (.cursor/rules/error-documentation.mdc): 
  - During your interaction, if you find a fix to a mistake in this project or a correction you received reusable, you should take note in the @error-documentation.mdc file so you will not make the same mistake again.
  - Known issues: their state, context, and resolution

  7. [lessons-learned.mdc](mdc:.cursor/rules/lessons-learned.mdc) (.cursor/rules/lessons-learned.mdc): learning journal for each project
  - It captures important patterns, preferences, and project intelligence
  - It is detailed in @lessons-learned.mdc

## Context Files (Optional)
Detailed docs. Retrieve on demand if needed for context.

1. docs/literature/ :
  - literature survey and researches are in this directory  
  - Each literature topic is a latex file (docs/literature/*.tex)

2. tasks/rfc/ :
  - contains RFC for each individual task in @tasks_plan.md
  - RFCs will be in latex file format (tasks/*.tex)

## Additional Context
Create additional files or folders as Memory files in docs/ or tasks/ when they help organize:
- Integration specifications
- Testing strategies
- Benchmarking setups
- Possible Extensions
- Deployment procedures

# Core Workflows
Now we define the procedural workflows to read/write to these memory files.
The system operates in distinct MODES: (PLAN/ACT) or analogously (Architect/Code), controlled exclusively by the user input or the task in current request. Current input will determine the MODE, based on which the Workflow selection is always dictated. In user input explicit mode setting can also be specified by "MODE = PLAN MODE"/"Architect MODE" or "MODE = ACT MODE"/"Code MODE", so if explicit MODE setting present follow that, else guess the mode from the request. Ask for the MODE if you are not 100% confident, if any doubt ask explicitly.

## PLAN or Architect MODE
```mermaid
flowchart TD
    Start[Start] --> ReadFiles[Read Memory Files ("docs/": (a)"docs/architecture.md" (b)"docs/product_requirement_docs.md" (c)"docs/technical.md". "tasks/" : (a)"tasks/active_context.md" (b)"tasks/tasks_plan.md". if needed further: "docs/literature" and "tasks/rfc") ]
    ReadFiles --> CheckFiles{Files Complete?}
    
    CheckFiles -->|No| Plan[Create Plan]
    Plan --> DocumentChat[Document in Chat]
    
    CheckFiles -->|Yes| VerifyContext[Verify Context]
    VerifyContext --> Strategy[Develop Strategy]
    Strategy --> Present[Present Approach]
    
    Present --> Verification{Approach Verified?}

    Verification -->|No| Clarify[Seek Clarification]
    Clarify --> Strategy[Develop Strategy]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bhartendu-Kumar/rules_template](https://github.com/Bhartendu-Kumar/rules_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
