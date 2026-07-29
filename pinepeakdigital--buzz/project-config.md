---
trigger: always_on
description: When you are assigned to an issue, **ALWAYS** check all comments on the issue before proceeding with implementation. This ensures you have the complete context and latest requirements.
---

# GitHub Copilot Instructions

## Check Issue Comments Before Implementation

When you are assigned to an issue, **ALWAYS** check all comments on the issue before proceeding with implementation. This ensures you have the complete context and latest requirements.

### How to Check Issue Comments

Use the GitHub MCP tools to retrieve all comments on the issue:

```
github-mcp-server-issue_read
  method: get_comments
  owner: <repo-owner>
  repo: <repo-name>
  issue_number: <issue-number>
```

### Important Guidelines

- **Read all comments** from newest to oldest to understand the full conversation
- Look for:
  - Clarifications of the original requirements
  - Additional context or edge cases to consider
  - User feedback or suggestions
  - Changes to the scope or priorities
  - Related issues or PRs that might be relevant
- If there are many comments, pay special attention to:
  - Comments from the issue author or repository maintainers
  - Recent comments (within the last few days)
  - Comments that explicitly mention implementation details or requirements
- **Ask for clarification** if comments contradict each other or if requirements are unclear
- Update your implementation plan based on insights from the comments

### Example Usage

For the repository `PinePeakDigital/buzz` with issue #123:

```
github-mcp-server-issue_read
  method: get_comments
  owner: PinePeakDigital
  repo: buzz
  issue_number: 123
```

This will return all comments on the issue, which you should review before starting implementation.

## Commit Message Format

This project uses [Conventional Commits](https://www.conventionalcommits.org/) for all commit messages. When suggesting commit messages, always follow this format:

```
type(scope): description
```

### Valid Commit Types

- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect the meaning of the code (formatting, whitespace, etc.)
- `refactor`: A code change that neither fixes a bug nor adds a feature
- `test`: Adding missing tests or correcting existing tests
- `chore`: Changes to the build process or auxiliary tools
- `perf`: A code change that improves performance
- `ci`: Changes to CI configuration files and scripts
- `build`: Changes that affect the build system or external dependencies
- `revert`: Reverts a previous commit

### Scope (Optional)

The scope is optional and should be the name of the affected module, component, or area of the codebase (e.g., `api`, `db`, `auth`, `ui`).

### Examples

Good commit messages:
```
feat: add user authentication system
fix(api): handle nil pointer in user service
docs: update README with installation steps
refactor(db): simplify connection pooling
test: add tests for beeminder API integration
chore: update dependencies
perf(grid): optimize goal rendering
ci: add linting to workflow
```

### Notes

- The description should be in lowercase and start with a verb in imperative mood
- Keep the description concise but descriptive
- A commit-msg hook enforces this format, so all commits must comply
- See `DEVELOPMENT.md` for more details on our git workflow

## Pull Request Title Format

Pull request titles should follow the same [Conventional Commits](https://www.conventionalcommits.org/) format as commit messages. This ensures consistency across the project and makes it easier to understand the purpose of each PR at a glance.

### Format

```
type(scope): description
```

### Guidelines

- Use the same commit types as listed above (`feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`, `revert`)
- The scope is optional but recommended for clarity
- Keep the description concise but descriptive
- Use lowercase and imperative mood (e.g., "add" not "adds" or "added")
- Don't end with a period

### Examples

Good PR titles:
```
feat: add goal archiving feature
fix(grid): prevent crash on empty goal list
docs: update authentication setup guide
refactor(beeminder): simplify API error handling
test: add integration tests for goal creation
chore: update dependencies to latest versions
```

Bad PR titles:
```
Added new feature (not imperative mood, no type)
Fix bug. (ends with period, missing scope/description)
Updates (too vague, not descriptive)
```

## Accessing CodeRabbit PR Feedback

When working on a PR branch, you can access all CodeRabbit feedback using the GitHub MCP tools available to Copilot. The GitHub MCP provides authenticated access to GitHub's API without requiring token configuration.

### What Feedback is Available

CodeRabbit provides feedback in three locations:

1. **General PR Comments** - Timeline comments posted by CodeRabbit on the PR conversation
2. **Inline Review Comments** - Code-specific comments attached to particular lines in the diff
3. **Review Summaries** - Overall review summaries submitted by CodeRabbit

### GitHub MCP Tools to Use

Use the following MCP tools in sequence to retrieve all CodeRabbit feedback:

#### Step 1: Find the Current PR

```
github-mcp-server-list_pull_requests
  owner: <repo-owner>
  repo: <repo-name>
  state: open
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PinePeakDigital/buzz](https://github.com/PinePeakDigital/buzz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
