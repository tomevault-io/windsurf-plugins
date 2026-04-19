---
trigger: always_on
description: <this-repo Guidelines>
---

<this-repo Guidelines>

# Guidelines to work in this project

- Use DDD to split the main domains of the project into separate folders
- Use TDD for unit tests using Vitest
- Do not repeat your types. Put the types closer to the domain and re-use them! Extract/pick types from other types for relationships
- Write small, composable, pure functions using DI to ease the tests
- Always make sure you code is formatted with Prettier
- Always make sure you code is linted with ESLint
- Always use Perplexity to search the web for best practices around the subject you are working on
- Always use Gemini to peer review the code you write
- Configurations must be environment variables, and you should never push your .env file (create a .env.sample with placeholder values)
- Do not explicit declare the extension of the files in the imports
- **ALWAYS** Work with your AI team in a feedback loop! Search, try, review, test, repeat.

<this-repo Integration>

<cursor-tools Integration>
# Instructions
Use the following commands to get AI assistance:

**Web Search:**
`cursor-tools web "<your question>"` - Get answers from the web using Perplexity AI (e.g., `cursor-tools web "latest weather in London"`)
when using web for complex queries suggest writing the output to a file somewhere like local-research/<query summary>.md.

**Repository Context:**
`cursor-tools repo "<your question>"` - Get context-aware answers about this repository using Google Gemini (e.g., `cursor-tools repo "explain authentication flow"`)

**Documentation Generation:**
`cursor-tools doc [options]` - Generate comprehensive documentation for this repository (e.g., `cursor-tools doc --output docs.md`)
when using doc for remote repos suggest writing the output to a file somewhere like local-docs/<repo-name>.md.

**GitHub Information:**
`cursor-tools github pr [number]` - Get the last 10 PRs, or a specific PR by number (e.g., `cursor-tools github pr 123`)
`cursor-tools github issue [number]` - Get the last 10 issues, or a specific issue by number (e.g., `cursor-tools github issue 456`)

**Stagehand Browser Automation:**
`cursor-tools browser open <url> [options]` - Open a URL and capture page content, console logs, and network activity (e.g., `cursor-tools browser open "https://example.com" --html`)
`cursor-tools browser act "<instruction>" --url=<url> [options]` - Execute actions on a webpage using natural language instructions (e.g., `cursor-tools browser act "Click Login" --url=https://example.com`)
`cursor-tools browser observe "<instruction>" --url=<url> [options]` - Observe interactive elements on a webpage and suggest possible actions (e.g., `cursor-tools browser observe "interactive elements" --url=https://example.com`)
`cursor-tools browser extract "<instruction>" --url=<url> [options]` - Extract data from a webpage based on natural language instructions (e.g., `cursor-tools browser extract "product names" --url=https://example.com/products`)

**Notes on Browser Commands:**

- All browser commands are stateless unless --connect-to is used to connect to a long-lived interactive session. In disconnected mode each command starts with a fresh browser instance and closes it when done.
- When using `--connect-to`, special URL values are supported:
  - `current`: Use the existing page without reloading
  - `reload-current`: Use the existing page and refresh it (useful in development)
- Multi step workflows involving state or combining multiple actions are supported in the `act` command using the pipe (|) separator (e.g., `cursor-tools browser act "Click Login | Type 'user@example.com' into email | Click Submit" --url=https://example.com`)
- Video recording is available for all browser commands using the `--video=<directory>` option. This will save a video of the entire browser interaction at 1280x720 resolution. The video file will be saved in the specified directory with a timestamp.
- DO NOT ask browser act to "wait" for anything, the wait command is currently disabled in Stagehand.

**Tool Recommendations:**

- `cursor-tools web` is best for general web information not specific to the repository.
- `cursor-tools repo` is ideal for repository-specific questions, planning, code review and debugging.
- `cursor-tools doc` generates documentation for local or remote repositories.
- `cursor-tools browser` is useful for testing and debugging web apps.

**Running Commands:**

Use `cursor-tools <command>` (if in PATH) or `npm exec cursor-tools "<command>"`, `yarn cursor-tools "<command>"`, `pnpm cursor-tools "<command>"`.

**General Command Options (Supported by all commands):**
--model=<model name>: Specify an alternative AI model to use.
--max-tokens=<number>: Control response length
--save-to=<file path>: Save command output to a file (in _addition_ to displaying it)
--help: View all available options (help is not fully implemented yet)

**Documentation Command Options:**
--from-github=<GitHub username>/<repository name>[@<branch>]: Generate documentation for a remote GitHub repository

**GitHub Command Options:**
--from-github=<GitHub username>/<repository name>[@<branch>]: Access PRs/issues from a specific GitHub repository

**Browser Command Options (for 'open', 'act', 'observe', 'extract'):**
--console: Capture browser console logs (enabled by default, use --no-console to disable)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/atreib) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
