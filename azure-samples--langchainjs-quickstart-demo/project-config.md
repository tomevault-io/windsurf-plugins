---
trigger: always_on
description: This is a generative AI demo application called "Ask YouTube" that showcases building RAG (Retrieval-Augmented Generation) applications using LangChain.js and Azure services. The application allows users to ask text-based questions about YouTube videos by analyzing their transcripts.
---

# AGENTS.md

## Project Overview

This is a generative AI demo application called "Ask YouTube" that showcases building RAG (Retrieval-Augmented Generation) applications using LangChain.js and Azure services. The application allows users to ask text-based questions about YouTube videos by analyzing their transcripts.

**Key Technologies:**
- **Language:** JavaScript (ES modules)
- **Runtime:** Node.js >= 20
- **Framework:** Azure Functions v4 with HTTP streaming support (preview feature)
- **AI Framework:** LangChain.js v0.3
- **Package Manager:** npm

**Architecture:**
The project has two implementation variants:
1. **Local Prototype** (`src/local.js`): Uses Ollama (LLaMa3) for completion, nomic-embed-text for embeddings, and FAISS for vector storage
2. **Azure Cloud Version** (`src/azure.js`): Uses Azure OpenAI (GPT-4 Turbo) for completion, text-embedding-3-large for embeddings, and Azure AI Search for vector storage

Both versions can be deployed as serverless APIs using Azure Functions runtime with HTTP streaming support.

## Setup Commands

### Prerequisites Installation
Before starting, ensure you have:
- Node.js version 20 or higher
- Ollama (for local development)
- Git

### Initial Setup

```bash
# Clone the repository
git clone <your-repo-url>
cd langchainjs-quickstart-demo

# Install dependencies
npm install

# For local development, pull required Ollama models
ollama pull llama3
ollama pull nomic-embed-text
```

### Environment Configuration

For Azure cloud version, create a `.env` file in the project root:

```bash
AZURE_AISEARCH_ENDPOINT=https://<your-service-name>.search.windows.net
AZURE_AISEARCH_KEY=<your-aisearch-key>
AZURE_OPENAI_API_KEY=<your-openai-key>
AZURE_OPENAI_API_ENDPOINT=<your-openai-endpoint>
AZURE_OPENAI_API_DEPLOYMENT_NAME="gpt-4-turbo"
AZURE_OPENAI_API_EMBEDDINGS_DEPLOYMENT_NAME="text-embedding-3-large"
AZURE_OPENAI_API_VERSION="2024-02-01"
```

**Important:** The `.env` file is gitignored and should never be committed.

## Development Workflow

### Running the Application

**Local Prototype (fastest way to start):**
```bash
npm run start:local
```

**Azure Cloud Version:**
```bash
npm run start:azure
```

**API Server (Azure Functions runtime):**
```bash
# Local version (default)
npm start

# Azure version
USE_AZURE=true npm start
```

### Testing the API

Once the server is running (default port: 7071), test with curl:

```bash
# Basic request
curl http://localhost:7071/api/ask \
  -H 'Content-Type: application/json' \
  -d '{ "question": "Will GPT-4 Turbo be available on Azure?" }'

# With streaming output (add -N flag)
curl -N http://localhost:7071/api/ask \
  -H 'Content-Type: application/json' \
  -d '{ "question": "What are the news about GPT-4 models?" }'
```

### Hot Reload / Watch Mode

The Azure Functions runtime automatically reloads when files change. No additional watch commands needed.

## Project Structure

```
├── src/
│   ├── local.js              # Local prototype using Ollama + FAISS
│   ├── azure.js              # Azure version using Azure OpenAI + AI Search
│   └── functions/
│       ├── ask.js            # Azure Functions HTTP trigger
│       └── lib/
│           ├── local.js      # Local RAG chain logic
│           └── azure.js      # Azure RAG chain logic
├── docs/                     # Extended documentation
├── package.json              # Dependencies and scripts
├── host.json                 # Azure Functions configuration
├── local.settings.json       # Local Functions settings
└── api.http                  # Sample HTTP requests
```

## Code Style Guidelines

### General Conventions
- **Module System:** ES modules (`import`/`export`)
- **Indentation:** 2 spaces (enforced by `.editorconfig`)
- **Line Endings:** LF (Unix-style)
- **Character Encoding:** UTF-8
- **Final Newline:** Always include

### JavaScript Patterns
- Use `const` and `let`, avoid `var`
- Use async/await for asynchronous operations
- Import from specific LangChain packages (e.g., `@langchain/openai`, `@langchain/community`)
- Import dotenv config at the top: `import "dotenv/config"`

### File Organization
- Keep RAG logic minimal and focused
- Separate local and Azure implementations clearly
- Azure Functions handlers in `src/functions/`
- Shared chain logic in `src/functions/lib/`

### LangChain.js Patterns
- Use `RecursiveCharacterTextSplitter` with `chunkSize: 1500` and `chunkOverlap: 200`
- Use `createStuffDocumentsChain` for document combination
- Use `ChatPromptTemplate.fromMessages` for prompt construction
- Standard retriever pattern: `vectorStore.asRetriever()`

## Testing Instructions

**Note:** This project currently has no automated test suite. Manual testing is required.

### Manual Testing Steps

1. **Test Local Version:**
   ```bash
   npm run start:local
   ```
   Verify:
   - Documents load from YouTube
   - Text splitting completes
   - Embeddings are generated
   - Vector store is populated
   - Question answering produces relevant results

2. **Test Azure Version:**
   ```bash
   npm run start:azure
   ```
   Verify:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/langchainjs-quickstart-demo](https://github.com/Azure-Samples/langchainjs-quickstart-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
