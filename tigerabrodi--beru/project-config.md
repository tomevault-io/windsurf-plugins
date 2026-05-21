---
trigger: always_on
description: Global Rule. This rule should ALWAYS be loaded.
---

cursor-tools is a CLI tool that allows you to interact with AI models and other tools.
cursor-tools is installed on this machine and it is available to you to execute. You're encouraged to use it.

<cursor-tools Integration>
# Instructions
Use the following commands to get AI assistance:

**Direct Model Queries:**
`cursor-tools ask "<your question>" --provider <provider> --model <model>` - Ask any model from any provider a direct question (e.g., `cursor-tools ask "What is the capital of France?" --provider openai --model o3-mini`). Note that this command is generally less useful than other commands like `repo` or `plan` because it does not include any context from your codebase or repository.

**Implementation Planning:**
`cursor-tools plan "<query>"` - Generate a focused implementation plan using AI (e.g., `cursor-tools plan "Add user authentication to the login page"`)
The plan command uses multiple AI models to:
1. Identify relevant files in your codebase (using Gemini by default)
2. Extract content from those files
3. Generate a detailed implementation plan (using OpenAI o3-mini by default)

**Plan Command Options:**
--fileProvider=<provider>: Provider for file identification (gemini, openai, anthropic, perplexity, modelbox, or openrouter)
--thinkingProvider=<provider>: Provider for plan generation (gemini, openai, anthropic, perplexity, modelbox, or openrouter)
--fileModel=<model>: Model to use for file identification
--thinkingModel=<model>: Model to use for plan generation
--debug: Show detailed error information

**Web Search:**
`cursor-tools web "<your question>"` - Get answers from the web using a provider that supports web search (e.g., Perplexity models and Gemini Models either directly or from OpenRouter or ModelBox) (e.g., `cursor-tools web "latest shadcn/ui installation instructions"`)
when using web for complex queries suggest writing the output to a file somewhere like local-research/<query summary>.md.

**Web Command Options:**
--provider=<provider>: AI provider to use (perplexity, gemini, modelbox, or openrouter)
--model=<model>: Model to use for web search (model name depends on provider)
--max-tokens=<number>: Maximum tokens for response

**Repository Context:**
`cursor-tools repo "<your question>" [--subdir=<path>]` - Get context-aware answers about this repository using Google Gemini (e.g., `cursor-tools repo "explain authentication flow"`). Use the optional `--subdir` parameter to analyze a specific subdirectory instead of the entire repository (e.g., `cursor-tools repo "explain the code structure" --subdir=src/components`)

**Documentation Generation:**
`cursor-tools doc [options]` - Generate comprehensive documentation for this repository (e.g., `cursor-tools doc --output docs.md`)
when using doc for remote repos suggest writing the output to a file somewhere like local-docs/<repo-name>.md.

**GitHub Information:**
`cursor-tools github pr [number]` - Get the last 10 PRs, or a specific PR by number (e.g., `cursor-tools github pr 123`)
`cursor-tools github issue [number]` - Get the last 10 issues, or a specific issue by number (e.g., `cursor-tools github issue 456`)

**ClickUp Information:**
`cursor-tools clickup task <task_id>` - Get detailed information about a ClickUp task including description, comments, status, assignees, and metadata (e.g., `cursor-tools clickup task "task_id"`)

**Model Context Protocol (MCP) Commands:**
Use the following commands to interact with MCP servers and their specialized tools:
`cursor-tools mcp search "<query>"` - Search the MCP Marketplace for available servers that match your needs (e.g., `cursor-tools mcp search "git repository management"`)
`cursor-tools mcp run "<query>"` - Execute MCP server tools using natural language queries (e.g., `cursor-tools mcp run "list files in the current directory" --provider=openrouter`). The query must include sufficient information for cursor-tools to determine which server to use, provide plenty of context.

The `search` command helps you discover servers in the MCP Marketplace based on their capabilities and your requirements. The `run` command automatically selects and executes appropriate tools from these servers based on your natural language queries. If you want to use a specific server include the server name in your query. E.g. `cursor-tools mcp run "using the mcp-server-sqlite list files in directory --provider=openrouter"`

**Notes on MCP Commands:**
- MCP commands require `ANTHROPIC_API_KEY` or `OPENROUTER_API_KEY` to be set in your environment
- By default the `mcp` command uses Anthropic, but takes a --provider argument that can be set to 'anthropic' or 'openrouter'
- Results are streamed in real-time for immediate feedback
- Tool calls are automatically cached to prevent redundant operations
- Often the MCP server will not be able to run because environment variables are not set. If this happens ask the user to add the missing environment variables to the cursor tools env file at ~/.cursor-tools/.env

**Stagehand Browser Automation:**
`cursor-tools browser open <url> [options]` - Open a URL and capture page content, console logs, and network activity (e.g., `cursor-tools browser open "https://example.com" --html`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigerabrodi/beru](https://github.com/tigerabrodi/beru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
