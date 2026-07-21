---
trigger: always_on
description: When responding to queries about this repository:
---

When responding to queries about this repository:

1. Suggest relevant commands from the "Essential Commands" section when applicable
2. Highlight Nx's focus on monorepos and its key features like smart task execution, code generation, and project graph
   analysis
3. Mention the plugin ecosystem and support for various frameworks when relevant
4. Emphasize the importance of running the full validation suite before committing changes

Always strive to provide accurate, helpful responses that align with the best practices and workflows described in this
file.

## Documentation Contributions

When working on Nx documentation, all documentation content lives in the `astro-docs/` folder. This is the new Astro-based documentation site built with Starlight.

**Important**: Before making any documentation changes, read the `astro-docs/README.md` file for detailed guidance on:

- Project structure and architecture
- Content types (regular docs, dynamic plugin docs, CLI docs)
- Available Markdoc tags for rich content
- Development workflow and commands
- Sidebar management

### Quick Reference

- Documentation content: `astro-docs/src/content/docs/`
- Use `.mdoc` (Markdoc) or `.mdx` format for documentation files
- Run `nx serve astro-docs` to start the local dev server
- Sidebar structure is defined in `astro-docs/sidebar.mts`

## GitHub Issue Response Mode

When responding to GitHub issues, determine your approach based on how the request is phrased:

### Plan-First Mode (Default)

Use this approach when users ask you to:

- "analyze", "investigate", "assess", "review", "examine", or "plan"
- Or when the request is ambiguous

In this mode:

1. Provide a detailed analysis of the issue
2. Create a comprehensive implementation plan
3. Break down the solution into clear steps
4. Then please post the plan as a comment on the issue

### Immediate Implementation Mode

Use this approach when users ask you to:

- "fix", "implement", "solve", "build", "create", "update", or "add"
- Or when they explicitly request immediate action

In this mode:

1. Analyze the issue quickly
2. Implement the complete solution immediately
3. Make all necessary code changes. Please make multiple commits so that the changes are easier to review.
4. Run appropriate tests and validation
5. If the tests, are not passing, please fix the issues and continue doing this up to 3 more times until the tests pass
6. Once the tests pass, push a branch and then suggest opening a PR which has a description of the changes made, and
   that
   it make sure that it explicitly says "Fixes #ISSUE_NUMBER" to automatically close the issue when the PR is merged.

## Avoid making changes to generated files

Files under `generated` directories are generated based on a different source file and should not be modified directly.
Find the underlying source and modify that instead.

## Essential Commands

### Code Formatting

After code changes are made, please make sure to format the files with prettier via `npx prettier -- FILE_NAME`

### Pre-push Validation

```bash
# Full validation suite - run before committing
nx prepush
```

If the prepush validation suite fails, please fix the issues before proceeding with your work. This ensures that all
code adheres to the project's standards and passes all tests. DO NOT make a new commit to fix these issues. Instead,
amend the current commit.

### Testing Changes in Other Repos

To test a locally built Nx package in another repository (e.g., to verify a fix end-to-end):

```bash
pnpm copy-built-package --package nx --repo ../path/to/test-repo
```

This builds the package and copies it into the target repo's `node_modules`. It works for all packages including native Rust code.

### Testing Changes

After code changes are made, first test the specific project where the changes were made:

```bash
nx run-many -t test,build,lint -p PROJECT_NAME
```

After verifying the individual project, validate that the changes in projects which have been affected:

```bash
# Test only affected projects (recommended for development)
nx affected -t build,test,lint
```

As the last step, run the e2e tests to fully ensure that changes are valid:

```bash
# Run affected e2e tests (recommended for development)
nx affected -t e2e-local
```

## Fixing GitHub Issues

When working on a GitHub issue, follow this systematic approach:

### 1. Get Issue Details

```bash
# Get issue details using GitHub CLI (replace ISSUE_NUMBER with actual number)
gh issue view ISSUE_NUMBER

# View multiple issues efficiently in one command
gh issue list --limit 50 --json number,title,state,labels,assignees,updatedAt,body --jq '.[] | select(.number == 123 or .number == 456 or .number == 789)'

# Or filter by specific criteria to get multiple related issues
gh issue list --label "bug" --state "open" --json number,title,body,labels --jq '.[]'
gh issue list --assignee "@me" --json number,title,body,state --jq '.[]'
```

**Tip**: Instead of running `gh issue view` multiple times, use `gh issue list` with JSON output and filtering to gather
information about multiple issues in a single command. This is much more efficient than viewing issues one at a time.

**Always provide clickable links**: When discussing GitHub issues or PRs, always include the full GitHub URL so the user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nrwl/nx](https://github.com/nrwl/nx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
