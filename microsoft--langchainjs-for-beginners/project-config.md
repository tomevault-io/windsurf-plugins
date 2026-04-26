---
trigger: always_on
description: This is **LangChain.js for Beginners** - a comprehensive educational course teaching AI application development with LangChain.js and TypeScript. The repository contains 8 sections (00-07) covering everything from basic chat models to autonomous agents, RAG systems, and Model Context Protocol (MCP) integration.
---

# AGENTS.md

## Project Overview

This is **LangChain.js for Beginners** - a comprehensive educational course teaching AI application development with LangChain.js and TypeScript. The repository contains 8 sections (00-07) covering everything from basic chat models to autonomous agents, RAG systems, and Model Context Protocol (MCP) integration.

**Architecture**: Educational course structure with 71+ runnable TypeScript examples organized by topic
**Key Technologies**: LangChain.js, TypeScript, tsx, Node.js >=22.0.0 (LTS), MCP, OpenAI/Microsoft Foundry/Anthropic
**Purpose**: Teaching developers how to build AI-powered applications using an agent-first philosophy
**Teaching Approach**: Agent-first (Tools → Agents → Documents → Agentic RAG)

## Repository Structure

```
LangChainJS-for-Beginners/
├── 00-course-setup/          # Initial setup and configuration
├── 01-introduction/          # LangChain.js basics
├── 02-chat-models/           # Chat models and interactions
├── 03-prompts-messages-outputs/ # Dynamic prompts, messages, and structured outputs
├── 04-function-calling-tools/   # Function calling and tool integration (Agent-first: Step 1)
├── 05-agents/                   # Autonomous agents (Agent-first: Step 2)
├── 06-mcp/                      # Model Context Protocol - connecting AI to external services
├── 07-documents-embeddings-semantic-search/ # Document processing, embeddings, semantic search (Agent-first: Step 3)
├── 08-agentic-rag-systems/      # Agentic RAG where agents decide when to search (Agent-first: Step 4)
│   └── samples/
│       └── mcp-rag-server/      # RAG as an MCP service (capstone example)
├── data/                     # Sample data files
├── scripts/                  # Build and validation scripts
│   ├── build-check.ts        # TypeScript compilation validator
│   ├── test-setup.ts         # Environment setup tester
│   └── validate-examples.ts  # Code example test runner
├── .env.example              # Environment configuration template
└── GLOSSARY.md               # Comprehensive course glossary
```

Each chapter contains:
- `README.md` - Conceptual explanations and learning content
- `assignment.md` - Hands-on exercises
- `code/` - Working code examples
- `solution/` - Assignment solutions
- `images/` - Supporting diagrams (some chapters)

## Setup Commands

### Initial Setup

```bash
# Install dependencies
npm install

# Copy environment template
cp .env.example .env

# Edit .env with your AI provider credentials
# Required: AI_API_KEY, AI_ENDPOINT, AI_MODEL, AI_EMBEDDING_MODEL
```

### AI Provider Configuration

The course supports multiple AI providers (no code changes needed - just update `.env`):

**GitHub Models** (Free - recommended for learning):
```bash
AI_API_KEY=your_github_personal_access_token
AI_ENDPOINT=https://models.inference.ai.azure.com
AI_MODEL=gpt-5-mini
AI_EMBEDDING_MODEL=text-embedding-3-small
```

**Microsoft Foundry** (Production):
```bash
AI_API_KEY=your_azure_openai_api_key
AI_ENDPOINT=https://your-resource.openai.azure.com/openai/v1
AI_MODEL=gpt-5-mini
AI_EMBEDDING_MODEL=text-embedding-3-small
```

**OpenAI Direct**:
```bash
AI_API_KEY=your_openai_api_key
AI_ENDPOINT=https://api.openai.com/v1
AI_MODEL=gpt-5-mini
AI_EMBEDDING_MODEL=text-embedding-3-small
```

### Environment Requirements

- Node.js >=22.0.0 (LTS - specified in `package.json`)
- npm (package manager)
- Valid AI provider credentials in `.env` file

## Development Workflow

### Running Individual Examples

```bash
# Run any TypeScript file directly with tsx
npx tsx 01-introduction/code/01-hello-world.ts

# Or use npm start
npm start 02-chat-models/code/01-multi-turn.ts

# Watch mode for development
npm run dev <file-path>
```

### Working on Course Content

When working on course content or examples:
1. Each chapter is self-contained with its own code examples
2. Code files use ES modules (`"type": "module"` in package.json)
3. All examples use environment variables for configuration (never hardcode keys)
4. Interactive examples should be marked in `validate-examples.ts`

## Testing Instructions

### Quick Build Check (Fast - 10-30 seconds)

```bash
# Type-check all TypeScript files without running them
npm run build
```

This compiles 71 TypeScript files to check for:
- ✅ Type errors
- ✅ Syntax errors
- ✅ Import issues
- ✅ No API calls - just compilation

### Full Validation (Comprehensive - 20-40 minutes)

```bash
# Run all code examples with actual API calls
npm test
```

**Important Notes**:
- Requires valid `AI_API_KEY`, `AI_ENDPOINT`, `AI_MODEL`, and `AI_EMBEDDING_MODEL` in `.env`
- Tests all examples sequentially to avoid rate limiting
- Interactive files are tested with automated input
- Some examples have 60-second timeouts (marked in `SLOW_FILES`)
- Examples in `future/` folder are not included in validation

### Testing Specific Examples

```bash
# Run a single example
npx tsx 03-prompt-templates/code/01-basic-template.ts

# Test a specific chapter
npx tsx 06-rag-systems/code/*.ts
```

### Test File Categories


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/langchainjs-for-beginners](https://github.com/microsoft/langchainjs-for-beginners) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
