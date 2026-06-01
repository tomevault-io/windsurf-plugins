---
trigger: always_on
description: This repository is a comprehensive learning course called "Generative AI for Beginners with JavaScript". It teaches developers how to integrate Generative AI into JavaScript applications through a time-traveling narrative where learners interact with historical figures.
---

# AGENTS.md

## Project Overview

This repository is a comprehensive learning course called "Generative AI for Beginners with JavaScript". It teaches developers how to integrate Generative AI into JavaScript applications through a time-traveling narrative where learners interact with historical figures.

**Key Technologies:**
- JavaScript/Node.js (ES Modules)
- TypeScript (for some lessons)
- OpenAI SDK
- Model Context Protocol (MCP)
- Express.js (for web applications)
- GitHub Models (free AI model access)
- Azure OpenAI (optional alternative)

**Architecture:**
- Multi-package learning repository (not a monorepo, but lesson-based structure)
- Each lesson in `/lessons/` contains standalone sample applications
- Main companion app in `/app/` directory for character interactions
- Video content and demos in `/videos/`
- Documentation in `/docs/`

## Repository Structure

```
/
├── app/              # Main companion app for interacting with historical characters
├── lessons/          # 8 lesson directories, each with sample code and solutions
│   ├── 01-intro-to-genai/
│   ├── 02-first-ai-app/
│   ├── 03-prompt-engineering/
│   ├── 04-structured-output/
│   ├── 05-rag/
│   ├── 06-tool-calling/
│   ├── 07-mcp/
│   └── 08-mcp-advanced/
├── docs/             # Setup guides and additional resources
├── videos/           # Video session content and demos
└── .devcontainer/    # GitHub Codespaces configuration
```

## Setup Commands

**Prerequisites:**
- Node.js 20 or higher
- GitHub account (for GitHub Models access)
- Optional: Personal Access Token (PAT) if running locally

**Initial Setup:**

```bash
# Clone the repository
git clone https://github.com/microsoft/generative-ai-with-javascript.git
cd generative-ai-with-javascript

# Install root dependencies
npm install

# For GitHub Codespaces users (recommended):
# The environment is pre-configured - no additional setup needed
# Use built-in GITHUB_TOKEN environment variable

# For local development:
# 1. Create a GitHub Personal Access Token (PAT) at https://github.com/settings/tokens
# 2. Set it as an environment variable: export GITHUB_TOKEN=your_token_here
```

**Running the Main Companion App:**

```bash
cd app
npm install
npm start
# Access at http://localhost:3000 or use "Open in Browser" in Codespaces
```

**Working with Individual Lessons:**

Each lesson has its own package.json. Navigate to the specific lesson directory:

```bash
cd lessons/02-first-ai-app/sample-app
npm install
npm start
```

## Development Workflow

### GitHub Codespaces (Recommended)

1. Fork the repository
2. Create a Codespace from your fork
3. Pre-configured environment includes:
   - Node.js 20
   - VSCode extensions (EditorConfig, Code Runner, REST Client)
   - Ollama with phi3 and all-minilm models
   - `GITHUB_TOKEN` automatically available
4. Navigate to any lesson or app directory and run `npm install && npm start`

### Local Development

1. Ensure Node.js 20+ is installed
2. Set `GITHUB_TOKEN` environment variable
3. Navigate to the directory you want to work with
4. Run `npm install` followed by `npm start`

### Environment Variables

- `GITHUB_TOKEN` - Required for GitHub Models API access
  - Automatically available in GitHub Codespaces
  - Must be set manually for local development
- No scopes/permissions needed for the token when using GitHub Models

## Working with Lessons

Each lesson follows a consistent structure:

- `README.md` - Main lesson content with narrative and technical instructions
- `sample-app/` or `code/` - Starting code for exercises
- `solution/` - Complete solution code
- `translations/` - Lesson translations in various languages

**To work on a lesson:**

```bash
cd lessons/<lesson-number>-<lesson-name>/sample-app
npm install
npm start
```

**For TypeScript-based lessons (MCP lessons 07-08):**

```bash
cd lessons/07-mcp/solution
npm install
npm run build    # Compiles TypeScript to build/ directory
npm run client   # Runs the MCP client
npm run inspect  # Runs MCP inspector tool
```

## Testing Instructions

**Current Testing Setup:**

This is primarily an educational repository focused on tutorial content. There are no automated unit or integration tests for the code samples.

**Manual Testing Approach:**

1. **Validate lesson code by running it:**
   ```bash
   cd lessons/<lesson-name>/sample-app
   npm install
   npm start
   ```

2. **Test the companion app:**
   ```bash
   cd app
   npm install
   npm start
   # Interact with characters through the web interface
   ```

3. **For MCP lessons, use the inspector tool:**
   ```bash
   cd lessons/07-mcp/solution
   npm run build
   npm run inspect
   # Test individual tools:
   npx @modelcontextprotocol/inspector --cli node build/index.js --method tools/call --tool-name add --tool-arg a=1 --tool-arg b=3
   ```

4. **Verify GitHub Models integration:**
   - Ensure `GITHUB_TOKEN` is set
   - Run any lesson sample that calls OpenAI APIs
   - Verify responses are generated successfully

## Code Style Guidelines

**JavaScript/TypeScript Conventions:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/generative-ai-with-javascript](https://github.com/microsoft/generative-ai-with-javascript) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
