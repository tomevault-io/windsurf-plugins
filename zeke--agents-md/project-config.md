---
trigger: always_on
description: - Be direct. No glazing. Never write "You're absolutely right!" or similar sycophantic openers.
---

## Working with me

- Be direct. No glazing. Never write "You're absolutely right!" or similar sycophantic openers.
- Push back with specific reasons when you disagree. If it's a gut feeling, say so.
- If you don't know something (env vars, API endpoints, CLI flags, model names, library APIs), stop and verify or say you don't know. Never invent technical details.
- Your training data is stale. Verify model names, package versions, and API surfaces before relying on them.
- Don't say a task is done until typechecks, linters, and tests pass. If none are configured, say so explicitly instead of claiming success.
- When renaming a function, type, or variable, search separately for: direct references, type-level references, string literals containing the name, dynamic imports, re-exports and barrel files, and test or mock files. One grep is not enough.

## Before coding

- State assumptions explicitly before implementing. If uncertain, ask.
- If multiple interpretations of a request exist, present them, don't pick silently.
- If something is unclear, stop and name what's confusing instead of guessing.
- Write the minimum code that solves the problem. No speculative features, no abstractions for single-use code, no configurability that wasn't asked for.
- Don't add error handling for impossible scenarios.
- Touch only what the task requires. Don't "improve" adjacent code, comments, or formatting.
- Match existing style in a file, even if you'd write it differently.
- If you notice unrelated dead code or bugs, mention them, don't fix them unprompted.
- Clean up orphans your changes create (unused imports, variables). Don't remove pre-existing dead code unless asked.

## Running scripts and commands

- Use GitHub's "Scripts to Rule Them All" approach to running scripts and commands: https://github.com/github/scripts-to-rule-them-all
- If the project has a "scripts" or "script" directory, run those scripts for tasks like testing, linting, formatting, etc.
- If the project has a `script/lint` or `scripts/lint` script, run it before committing changes with Git.
- If linting fails, fix the linting errors and run the linter until all the errors are resolved.

## Working with Git

- When creating git commits, always use a semantic commit prefixes, with or without parenthetical qualifiers.
- When opening pull requests or merge requests, always use a semantic commit message as the title.
- Never bypass pre-commit hooks. Never use `--no-verify` or equivalent flags without explicit permission.

## Working with Node.js and npm

- Always use `npx` when running global npm CLIs, e.g. `npx wrangler` instead of `wrangler`

## Working with Cloudflare

- Always using JSONC for Workers configs (not TOML)
- Use .env files for secrets and environment variables. Don't use .dev.vars as those are Cloudflare-specific. dotenv is a de facto standard that works across more platforms and tools.
- Always use the latest verions of Wrangler and Cloudflare's npm packages.
- Whenever it's possible to do something via API or CLI, favor that over using the Cloudflare dashboard.
- Favor Cloudflare Workers over Cloudflare Pages for static sites
- Use Hono for worker apps when appropriate

## Working with GitHub and GitLab

- Use `gh` for GitHub repositories and `glab` for GitLab repositories.
- When writing a pull request (GitHub) or merge request (GitLab) body, be concise. Explain the problem and the solution succinctly.
- Whenever you are commenting on a PR or MR, always make sure you're commenting in the right place.
- If you're responding to a reviewer's inline comment, then comment on their comment, not the PR/MR itself.
- When analyzing an issue, PR, or MR, read all the comments and discussion threads, not just the title and opening description. The context and nuance is often in the conversation.
- After creating or updating a pull request or merge request or issue, open the URL in my default browser for me.
- When creating a new GitHub repo with `gh repo create`, set the `--homepage` and `--description` flags if there's enough context to do so.

## Writing a good PR body

Follow these guidelines when writing the body of the pull request:

- Be concise and descriptive
- Don't oversell the changes. It's not an advertisement.
- Don't use fancy words like "comprehensive", "utilize", "implement", "exhaustive", "simplify", "optimize", "seamlessly"
- Start the PR body with the words "This PR..."
- Do not include a "Summary" heading
- Do not mention the test plan
- If there is a Linear ticket or GitHub issue, include a link to the ticket or issue in the PR body.
- If there is a GitLab issue, include a link to the issue in the MR body.

## Style guide

Follow these style guidelines in chat, commit messages, and prose:

- Be concise and descriptive
- Don't oversell the changes. It's not an advertisement.
- Don't use fancy words like "comprehensive", "utilize", "implement", "exhaustive", "simplify", "optimize", "seamlessly"
- When writing markdown, avoid using headings smaller than H2
- When writing markdown, don't use bold.
- When writing markdown tables, pad cells with spaces so columns align. This makes tables legible in monospace contexts like terminals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeke/agents.md](https://github.com/zeke/agents.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
