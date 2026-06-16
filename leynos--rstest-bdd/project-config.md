---
trigger: always_on
description: - **Code is for humans.** Write code with clarity and empathy—assume a
---

# Assistant instructions

## Code style and structure

- **Code is for humans.** Write code with clarity and empathy—assume a
  tired teammate will need to debug it at 3 a.m.
- **Comment *why*, not *what*.** Explain assumptions, edge cases, trade-offs, or
  complexity. Don't echo the obvious.
- **Clarity over cleverness.** Be concise, but favour explicit over terse or
  obscure idioms. Prefer code that's easy to follow.
- **Use functions and composition.** Avoid repetition by extracting reusable
  logic. Prefer generators or comprehensions, and declarative code to
  imperative repetition when readable.
- **Small, meaningful functions.** Functions must be small, clear in purpose,
  single responsibility, and obey command/query segregation.
- **Clear commit messages.** Commit messages should be descriptive, explaining
  what was changed and why.
- **Name things precisely.** Use clear, descriptive variable and function names.
  For booleans, prefer names with `is`, `has`, or `should`.
- **Structure logically.** Each file should encapsulate a coherent module. Group
  related code (e.g., models + utilities + fixtures) close together.
- **Group by feature, not layer.** Colocate views, logic, fixtures, and helpers
  related to a domain concept rather than splitting by type.
- **Use consistent spelling and grammar.** Comments must use en-GB-oxendict
  ("-ize" / "-yse" / "-our") spelling and grammar, with the exception of
  references to external APIs.
- **Illustrate with clear examples.** Function documentation must include clear
  examples demonstrating the usage and outcome of the function. Test
  documentation should omit examples where the example serves only to reiterate
  the test logic.
- **Keep file size manageable.** No single code file may be longer than 400
  lines.  Long switch statements or dispatch tables should be broken up by
  feature and constituents colocated with targets. Large blocks of test data
  should be moved to external data files.

## Documentation maintenance

- **Reference:** Use the markdown files within the `docs/` directory as a
  knowledge base and source of truth for project requirements, dependency
  choices, and architectural decisions. Start with
  [documentation contents](docs/contents.md) and
  [repository layout](docs/repository-layout.md) when orienting within the
  project.
- **Update:** When new decisions are made, requirements change, libraries are
  added/removed, or architectural patterns evolve, **proactively update** the
  relevant file(s) in the `docs/` directory to reflect the latest state. Ensure
  the documentation remains accurate and current.
- **Design decisions:** Record design decisions in the relevant design
  document. When a decision is substantive, capture it in an Architectural
  Decision Record (ADR) following the documentation style guide, and reference
  that ADR from the design document.
- **User-facing behaviour:** Update [users' guide](docs/users-guide.md) for
  behaviour or user-interface changes that users should know about.
- **Internal interfaces:** Document internally facing interfaces in the
  relevant component architecture document. Document internally facing
  conventions and practices in [developers' guide](docs/developers-guide.md).
- **Style:** All documentation must adhere to the
  [documentation style guide](docs/documentation-style-guide.md).

## Change quality & committing

- **Atomicity:** Aim for small, focused, atomic changes. Each change (and
  subsequent commit) should represent a single logical unit of work.
- **Quality Gates:** Before considering a change complete or proposing a commit,
  ensure it meets the following criteria:
  - New functionality or changes in behaviour are fully validated by relevant
    unit tests and behavioural tests.
  - Where a bug is being fixed, a unit test must be provided that demonstrates
    the corrected behaviour and validates the fix to guard against regression.
  - Passes all relevant unit and behavioural tests according to the guidelines
    above.
  - Passes lint checks
  - Adheres to formatting standards tested using a formatting validator.
- **Committing:**
  - Only changes that meet all the quality gates above should be committed.
  - Write clear, descriptive commit messages summarizing the change, following
    these formatting guidelines:
    - **Imperative Mood:** Use the imperative mood in the subject line (e.g.,
      "Fix bug", "Add feature" instead of "Fixed bug", "Added feature").
    - **Subject Line:** The first line should be a concise summary of the change
      (ideally 50 characters or fewer).
    - **Body:** Separate the subject from the body with a blank line. Subsequent
      lines should explain the *what* and *why* of the change in more detail,
      including rationale, goals, and scope. Wrap the body at 72 characters.
    - **Formatting:** Use Markdown for any formatted text (like bullet points or
      code snippets) within the commit message body.
  - Do not commit changes that fail any of the quality gates.

## Refactoring heuristics & workflow

- **Recognizing Refactoring Needs:** Regularly assess the codebase for potential
  refactoring opportunities. Perform refactoring when observing:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leynos/rstest-bdd](https://github.com/leynos/rstest-bdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
