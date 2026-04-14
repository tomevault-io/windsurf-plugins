---
trigger: always_on
description: - name: Development Philosophy
---

# ~/DevRules/.cursorrules

rules:

- name: Development Philosophy
  pattern: "\*_/_.go"
  content: |

  - Prefer small, incremental, compiling commits over large "big bang" changes.
  - Study and learn from existing code before writing new functionality.
  - Choose pragmatic, clear, and maintainable code over clever abstractions.
  - Each function or struct should have one clear responsibility.
  - Avoid premature abstractions or unnecessary complexity.
  - When in doubt, choose the simplest solution that clearly conveys intent.

- name: Implementation Workflow
  pattern: "\*_/_.go"
  content: |

  - Follow a test-driven flow when practical: red → green → refactor.
  - Break complex work into documented stages in `IMPLEMENTATION_PLAN.md`.
  - Every commit must:
    - Compile successfully
    - Pass all tests
    - Include new tests for new functionality
    - Conform to project formatting/linting
  - Commit messages should explain _why_ changes are made.

- name: Problem-Solving Guidelines
  pattern: "\*\*"
  content: |

  - Maximum 3 failed attempts before reassessing approach.
  - Document what failed, why, and alternatives you explored.
  - When multiple valid approaches exist, prioritize:
    1. Testability
    2. Readability
    3. Consistency
    4. Simplicity
    5. Reversibility

- name: Architecture & Code Quality
  pattern: "\*_/_.go"
  content: |

  - Use composition and interfaces, not inheritance or singletons.
  - Keep dependencies explicit; avoid hidden coupling.
  - Fail fast with descriptive errors that include context.
  - Never disable tests — fix or adapt them.
  - Write clear, idiomatic Go with descriptive naming and comments when necessary.

- name: Testing & Quality Gates
  pattern: "tests/\*_/_.go"
  content: |

  - Test behavior, not implementation details.
  - Keep tests deterministic and isolated.
  - One clear assertion per test where possible.
  - Follow existing testing utilities and conventions.
  - Definition of Done:
    - Tests passing
    - Code linted and formatted
    - No TODOs without issue references
    - Implementation matches the documented plan

- name: Documentation
  pattern: "\*\*"
  content: |

  - Use Mermaid diagrams to visualize complex flows, architecture, or relationships.
  - Prefer diagrams over lengthy textual descriptions when they add clarity.
  - Common diagram types: flowcharts, sequence diagrams, class diagrams, state diagrams.
  - Keep diagrams simple and focused on one concept at a time.
  - Always accompany diagrams with brief explanatory text.

- name: Development Safety
  pattern: "\*\*"
  content: |
  - Never use `--no-verify` to bypass hooks.
  - Never commit broken or uncompiled code.
  - Always document implementation stages as you progress.
  - Avoid introducing new tools unless justified.
  - Continuously learn from similar patterns in the codebase.

# Go layout rule

- name: Go File Layout & Organization
  pattern: "\*_/_.go"
  content: |

  - Organize Go source files from general → specific:

    1. Package declaration
    2. Imports
    3. File-level constants related to the file’s domain
    4. Type declarations (interfaces before structs)
    5. Constructor functions placed immediately after their type
    6. Methods on a type, grouped and ordered logically
    7. Standalone helper functions at the bottom

  - Keep all methods for a struct grouped together; do not scatter them.
  - Avoid alphabetically sorting methods; instead order them by conceptual flow (constructor → configuration → execute → cleanup).
  - Prefer conceptual cohesion: if a file becomes a “junk drawer” of unrelated types or helpers, split it by domain (e.g. `model.go`, `service.go`, `handlers.go`).
  - Constants should be placed near the top only if used broadly; otherwise, keep them close to the type they belong to.
  - When generating new code, ensure the AI places the new type and its constructor and methods together rather than scattered across the file.

# Temporal – Go

- name: Temporal Workflows - Go (Durable Execution & Determinism)
  pattern: "\*_/_.go"
  content: |

  - When editing Go code that imports `go.temporal.io/sdk/workflow`, treat it as Workflow Definition code and follow Temporal's determinism constraints.

  - Avoid intrinsic non-deterministic logic inside Workflows:

    - Do not use `math/rand` (or other non-deterministic RNG) to decide behavior in a Workflow; move such logic into Activities or use deterministic alternatives.
    - Do not call external systems (HTTP APIs, DB queries, file I/O, other services) directly from Workflows; call Activities instead.
    - Do not rely on system time (`time.Now`, `time.Sleep`) in Workflows.
      - Use `workflow.Now()` and `workflow.Sleep()` instead so time is driven by the Event History and remains deterministic.
    - Do not use native goroutines, channels, or `select` inside Workflows.
      - Use `workflow.Go()`, `workflow.Channel()`, and `workflow.Selector()` for concurrency and coordination.
    - Avoid iterating over Go maps with `range` where ordering matters; sort keys or offload to Activities if ordering is important.

  - Do not store or branch on Workflow run IDs inside Workflow logic; that can create nondeterministic branching between runs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhou-en) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
