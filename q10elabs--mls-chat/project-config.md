---
trigger: always_on
description: provides an overview of the main items defined in that file. Update
---

# CLAUDE.md - Repository Rules and Guidelines

## Agent session persistence and context tracking

You must always create, update maintain a changelog file that tracks specifications, changes, decisions, and progress.
Do this also at the beginning of a task before searching any file or asking clarifying questions.

Update the file before and throughout a task to:

- Track specifications from the user
- Maintain awareness of ongoing tasks and implementation decisions
- Reference previous conversations through changelog files when relevant
- Track project evolution and architectural decisions over time

The file is placed in the `changelog/` directory with the naming pattern:

- **Format:** `YYYYMMDD-topic.md` (generate the timestamp using the shell command `date +%Y%m%d`)
- **Topic generation:** Auto-generate from the user's initial request
- **Example:** `20250814-claude-md-improvements.md`

The changelog file must include:

1. **Task Specification**: Clear description of the original request and scope
2. **High-Level Decisions**: Major architectural, technical, or strategic decisions made
3. **Requirements Changes**: Track when and how requirements are modified mid-conversation
4. **Files Modified**: List of all files created, modified, or deleted (no code diffs, just summaries)
5. **Rationales and Alternatives**: Why certain approaches were chosen over others
6. **Obstacles and Solutions**: Problems encountered and brief (1-line) solutions
7. **Current Status**: Progress tracking and next steps

Content Guidelines:

- **Include**: Decision rationales, file modification summaries, requirement changes, obstacles with solutions
- **Exclude**: Specific code diffs, redundant information, overly technical implementation details
- **Structure**: Flexible format optimized for the specific conversation type
- **Persistence**: Never delete changelog files after work completion

## High-level map of the repository structure for quick context

- `client/rust`: client using Rust
- `server`: server program (rust)

## Product vision for the project

This project aims to demonstrate how to use OpenMLS to implement a
simple multi-user chat application in the terminal.

See the file `README.md` for details.

## Require clarification and plan approval before making code changes

Before making any code changes other than the changelog, you must follow this two-step process:

### Step 1: Ask Clarifying Questions
- Always ask at least one clarifying question about the user's request
- Understand the full scope and context of what they're asking for
- Clarify any ambiguous requirements or edge cases
- Ask about preferred approaches if multiple solutions exist
- Confirm the expected behavior and user experience

### Step 2: Present Implementation Plan
- After receiving clarification, present a detailed implementation plan
- Break down the work into specific, actionable steps
- Identify which files will be created, modified, or deleted
- Explain the technical approach and any architectural decisions
- Highlight any potential risks, trade-offs, or dependencies
- Estimate the complexity and scope of changes
- **Wait for explicit user approval** before proceeding with any code changes

### Approval Requirements
- User must explicitly approve the plan with words like "yes", "approved", "proceed", "go ahead", or similar
- If the user suggests modifications to the plan, incorporate them and seek re-approval
- Do not assume silence or ambiguous responses mean approval

### Exceptions
- This process may be skipped only for trivial changes like fixing obvious typos or formatting
- When in doubt, always follow the full process rather than assuming an exception applies

### Example Flow
1. User: "Add a login form to the app"
2. Assistant: "I'd like to clarify a few things about the login form: [questions]"
3. User: [provides answers]
4. Assistant: "Based on your requirements, here's my implementation plan: [detailed plan]. Does this approach look good to you?"
5. User: "Yes, that looks good"
6. Assistant: [proceeds with implementation]


## Run linters after reporting changes completes

Before telling the user your work is complete, do:

- run `cargo fmt` on the Rust directories where you have performed code changes.
- run `cargo clippy --all-targets` to find linter warnings.

## Git workflow and commit practices including commit message formatting

Git Operation Rules:

- **User-initiated only**: Perform git operations only when explicitly prompted by the user
- **No automatic staging**: Never add files to the git index; always prompt the user to stage files manually
- **Command suggestions**: Provide exact git commands for the user to execute
- **Branch management**: User manages all branching operations manually

Commit Message Structure:

When prompted to generate commit messages, use this three-section format:

```
First line: [one line summary of change]

Previous: [Feature-specific description of the state before changes,
written as multi-line paragraphs describing what existed and how it
worked, focusing on the functionality being modified]

Changed: [High-level summary derived from `git diff --cached`,
describing what was modified, added, or removed in terms that connect

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [q10elabs/mls-chat](https://github.com/q10elabs/mls-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
