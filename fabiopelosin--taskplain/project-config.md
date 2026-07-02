---
trigger: always_on
description: This guide outlines how AI agents should work within the Taskplain project, covering workflow expectations, documentation practices, and the Taskplain task management system.
---

# Agent Operating Guide — Taskplain

This guide outlines how AI agents should work within the Taskplain project, covering workflow expectations, documentation practices, and the Taskplain task management system.

## Workflow Expectations

**Documentation and Task Management:**

- Keep README and all documentation (see Documentation Hygiene below) up to date with shipped behavior
- Update the task you're working on as you progress: status, priority, details, dependencies, parent-owned `children` lists, and all other metadata
- When you finish a task, ensure the task file reflects its final state and run `node dist/cli.js validate`
- Surface blockers or ambiguities by updating task narratives or acceptance criteria so the next agent has full context

**Task Lifecycle:**

- Treat `tasks/00-idea` directory as the idea inbox: clarify scope, fill every task heading, set dispatch metadata (`size`, `ambiguity`, `executor`, `isolation`, `touches`), and record dependencies before promoting work to `ready`
- Work incrementally with commits that clearly reference the task being addressed (see Git Commit Convention below)

**Multi-Agent Considerations:**

- Multiple agents may be working on the codebase concurrently
- If you find dirty files in git (uncommitted changes), ignore them—don't modify or delete them

## Documentation Hygiene

Always update these files as needed:

- [README.md](README.md) — project overview, installation, quick start
- [docs/product.md](docs/product.md) — product requirements and vision
- [docs/architecture.md](docs/architecture.md) — system design and technical decisions
- [docs/tech.md](docs/tech.md) — canonical tech stack, environment setup, dependency catalog, and day-to-day tooling practices.
- [docs/changelog.md](docs/changelog.md) — release history and changes
- [docs/decisions.md](docs/decisions.md) — architectural decision records (ADRs)

## Git Commit Convention

This project follows the [Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/) specification.

**Format:** `<type>(<scope>): <description>  [Task:<id>]`

**Examples:**

- `feat(cli): add list command with JSON output  [Task:cli-list]`
- `fix(validation): handle null values in metadata  [Task:validate-fix]`
- `docs(readme): update installation instructions  [Task:readme-polish]`
- `refactor(parser): extract markdown parsing logic  [Task:parser-refactor]`

**Common types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`, `revert`

Always include the `[Task:<id>]` trailer to link commits to their corresponding task.

## Taskplain usage

<!-- taskplain:start v0.2.0 -->
This repo uses Taskplain.

All work must flow through the Taskplain CLI to keep task history in-repo and deterministic. Verify installation: `taskplain --help`

**Core Workflow**

1. **Find work**: Search with `taskplain list --search "keyword" --output json` or check ready tasks with `taskplain list --state ready --output json`
2. **Start task**: Always run `taskplain pickup <id>` first—this verifies readiness, promotes parent tasks if needed, advances the task to in-progress, and bundles all ancestor context
3. **During work**:
   - Update acceptance criteria checkboxes (`- [ ] ...`) every few minutes to show progress
   - Keep task metadata current after each logical change
   - Ensure acceptance criteria are complete before starting—fix gaps immediately
4. **Finish task**: When all checkboxes are done:
   - Run `taskplain update <id> --meta commit_message="feat(scope): … [Task:<id>]"` to store the final subject.
   - Populate the Post-Implementation Insights stubs **before** completing. The default template in `src/docsources/task-template.md` already includes `## Post-Implementation Insights` with `### Changelog`, `### Decisions`, and `### Technical Changes`—keep those headings and add bullets beneath them. Cap the Technical Changes bullets at roughly ten lines so reviewers can scan quickly.
   - If you draft insights in a scratch file, load them with `taskplain update <id> --field post_implementation_insights @/tmp/insights.md`. The scratch file should only contain the subsection headings and entries—do **not** re-add the `## Post-Implementation Insights` heading. For example:
     ```md
     ### Changelog
     - Added API contract checks for metadata updates.

     ### Decisions
     - Reused existing validation helpers; deferred DTO refactor.

     ### Technical Changes
     - `src/services/validationService.ts`: Accepts Technical Changes headings when validating tasks.
     - `docs/cli-playbook.md`: Documented the ≤10 line expectation for Technical Changes summaries.
     ```
   - Run `taskplain complete <id> --check-acs` whenever you want the CLI to auto-check any remaining acceptance criteria boxes before finalizing. Skip the flag if you need to leave unchecked boxes for follow-up notes. Commit with `[Task:<id>]` trailer once the task is complete
5. **Validate**: Run `taskplain validate` after manual edits to prevent schema violations

**Critical Rules**

- Work on one task at a time. Handle multiple requests sequentially.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabiopelosin/taskplain](https://github.com/fabiopelosin/taskplain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
