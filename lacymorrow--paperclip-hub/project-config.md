---
trigger: always_on
description: Global Rule. This rule should ALWAYS be loaded.
---

vibe-tools is a CLI tool that allows you to interact with AI models and other tools.
vibe-tools is installed on this machine and it is available to you to execute. You're encouraged to use it.

<vibe-tools Integration>
# Instructions
Use the following commands to get AI assistance:

**Direct Model Queries:**
`vibe-tools ask "<your question>" --provider <provider> --model <model>` - Ask any model from any provider a direct question (e.g., `vibe-tools ask "What is the capital of France?" --provider openai --model o3-mini`). Note that this command is generally less useful than other commands like `repo` or `plan` because it does not include any context from your codebase or repository.
Note: in general you should not use the ask command because it does not include any context - other commands like `doc`, `repo`, or `plan` are usually better. If you are using it, make sure to include in your question all the information and context that the model might need to answer usefully.

**Ask Command Options:**
--provider=<provider>: AI provider to use (openai, anthropic, perplexity, gemini, modelbox, or openrouter)
--model=<model>: Model to use (required for the ask command)
--reasoning-effort=<low|medium|high>: Control the depth of reasoning for supported models (OpenAI o1/o3-mini models and Claude 3.7 Sonnet). Higher values produce more thorough responses for complex questions.

**Implementation Planning:**
`vibe-tools plan "<query>"` - Generate a focused implementation plan using AI (e.g., `vibe-tools plan "Add user authentication to the login page"`)
The plan command uses multiple AI models to:
1. Identify relevant files in your codebase (using Gemini by default)
2. Extract content from those files
3. Generate a detailed implementation plan (using OpenAI o3-mini by default)

**Plan Command Options:**
--fileProvider=<provider>: Provider for file identification (gemini, openai, anthropic, perplexity, modelbox, or openrouter)
--thinkingProvider=<provider>: Provider for plan generation (gemini, openai, anthropic, perplexity, modelbox, or openrouter)
--fileModel=<model>: Model to use for file identification
--thinkingModel=<model>: Model to use for plan generation

**Web Search:**
`vibe-tools web "<your question>"` - Get answers from the web using a provider that supports web search (e.g., Perplexity models and Gemini Models either directly or from OpenRouter or ModelBox) (e.g., `vibe-tools web "latest shadcn/ui installation instructions"`)
Note: web is a smart autonomous agent with access to the internet and an extensive up to date knowledge base. Web is NOT a web search engine. Always ask the agent for what you want using a proper sentence, do not just send it a list of keywords. In your question to web include the context and the goal that you're trying to acheive so that it can help you most effectively.
when using web for complex queries suggest writing the output to a file somewhere like local-research/<query summary>.md.

**Web Command Options:**
--provider=<provider>: AI provider to use (perplexity, gemini, modelbox, or openrouter)

**Repository Context:**
`vibe-tools repo "<your question>" [--subdir=<path>] [--from-github=<username/repo>]` - Get context-aware answers about this repository using Google Gemini (e.g., `vibe-tools repo "explain authentication flow"`). Use the optional `--subdir` parameter to analyze a specific subdirectory instead of the entire repository (e.g., `vibe-tools repo "explain the code structure" --subdir=src/components`). Use the optional `--from-github` parameter to analyze a remote GitHub repository without cloning it locally (e.g., `vibe-tools repo "explain the authentication system" --from-github=username/repo-name`).

**Documentation Generation:**
`vibe-tools doc [options]` - Generate comprehensive documentation for this repository (e.g., `vibe-tools doc --output docs.md`)
when using doc for remote repos suggest writing the output to a file somewhere like local-docs/<repo-name>.md.

**YouTube Video Analysis:**
`vibe-tools youtube "<youtube-url>" [question] [--type=<summary|transcript|plan|review|custom>]` - Analyze YouTube videos and generate detailed reports (e.g., `vibe-tools youtube "https://youtu.be/43c-Sm5GMbc" --type=summary`)
Note: The YouTube command requires a `GEMINI_API_KEY` to be set in your environment or .vibe-tools.env file as the GEMINI API is the only interface that supports YouTube analysis.

**GitHub Information:**
`vibe-tools github pr [number]` - Get the last 10 PRs, or a specific PR by number (e.g., `vibe-tools github pr 123`)
`vibe-tools github issue [number]` - Get the last 10 issues, or a specific issue by number (e.g., `vibe-tools github issue 456`)

**ClickUp Information:**
`vibe-tools clickup task <task_id>` - Get detailed information about a ClickUp task including description, comments, status, assignees, and metadata (e.g., `vibe-tools clickup task "task_id"`)

**Model Context Protocol (MCP) Commands:**
Use the following commands to interact with MCP servers and their specialized tools:
`vibe-tools mcp search "<query>"` - Search the MCP Marketplace for available servers that match your needs (e.g., `vibe-tools mcp search "git repository management"`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacymorrow/paperclip-hub](https://github.com/lacymorrow/paperclip-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
