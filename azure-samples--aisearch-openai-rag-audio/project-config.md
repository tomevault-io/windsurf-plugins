---
trigger: always_on
description: This file contains instructions for developers and automated coding agents working on the VoiceRAG application. It covers code layout, environment setup, testing procedures, and development conventions.
---

# Instructions for Coding Agents

This file contains instructions for developers and automated coding agents working on the VoiceRAG application. It covers code layout, environment setup, testing procedures, and development conventions.

## Overview

VoiceRAG is an application pattern demonstrating RAG (Retrieval Augmented Generation) with voice interfaces using Azure AI Search and the GPT-4o Realtime API for Audio. The application enables voice-based interactions with a knowledge base, with audio input processed through the browser and sent to Azure OpenAI's real-time API for responses.

**Main technologies:**
- **Backend**: Python 3.11+ with aiohttp web framework
- **Frontend**: React with TypeScript, built with Vite
- **Infrastructure**: Azure Bicep templates for Azure Container Apps deployment
- **Key Azure Services**: Azure OpenAI (GPT-4o Realtime API), Azure AI Search, Azure Container Apps
- **Deployment**: Azure Developer CLI (azd)

**Primary entry points:**
- `app/backend/app.py` - Backend application server (aiohttp)
- `app/frontend/src/main.tsx` - Frontend React application
- `scripts/start.sh` or `scripts/start.ps1` - Development server startup scripts

## Code layout

- `app/` - Main application code
  - `app/backend/` - Python backend code
    - `app/backend/app.py` - Main entry point for backend server (aiohttp)
    - `app/backend/rtmt.py` - Real-time middle tier for Azure OpenAI integration
    - `app/backend/ragtools.py` - RAG tools for Azure AI Search integration
    - `app/backend/setup_intvect.py` - Setup script for integrated vectorization
    - `app/backend/requirements.txt` - Python dependencies
  - `app/frontend/` - React TypeScript frontend
    - `app/frontend/src/` - Source code for React components
    - `app/frontend/src/locales/` - Translation files (en, es, fr, ja)
    - `app/frontend/package.json` - Node.js dependencies
    - `app/frontend/vite.config.ts` - Vite build configuration
  - `app/Dockerfile` - Container image definition for deployment
- `infra/` - Bicep infrastructure-as-code templates
  - `infra/main.bicep` - Main infrastructure definition
  - `infra/main.parameters.json` - Template parameters
  - `infra/core/` - Reusable Bicep modules
- `scripts/` - Helper scripts for development and deployment
  - `scripts/start.sh` / `scripts/start.ps1` - Start development server
  - `scripts/write_env.sh` / `scripts/write_env.ps1` - Generate .env file from azd
  - `scripts/setup_intvect.sh` / `scripts/setup_intvect.ps1` - Setup integrated vectorization
  - `scripts/load_python_env.sh` / `scripts/load_python_env.ps1` - Create Python virtual environment
- `data/` - Sample data files (Markdown documents)
- `docs/` - Documentation
  - `docs/existing_services.md` - Connect to existing Azure services
  - `docs/customizing_deploy.md` - Customize deployment options
  - `docs/manual_setup.md` - Manual setup instructions
- `.github/workflows/` - GitHub Actions workflows
  - `.github/workflows/template-validation.yaml` - Template validation workflow
  - `.github/workflows/azure-dev.yml` - Azure deployment workflow
- `azure.yaml` - Azure Developer CLI (azd) configuration
- `pyproject.toml` - Python project configuration (ruff linting rules)

## Running the code

### Prerequisites

Install the required tools:
- [Azure Developer CLI](https://aka.ms/azure-dev/install)
- [Node.js](https://nodejs.org/) (for frontend)
- [Python 3.11 or higher](https://www.python.org/downloads/)
- [Git](https://git-scm.com/downloads)

**Important for Windows users:**
- Python and pip must be in PATH
- [PowerShell](https://learn.microsoft.com/powershell/scripting/install/installing-powershell) is required

### Local development setup

1. **Create and activate Python virtual environment:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

2. **Install Python dependencies:**
   ```bash
   pip install -r app/backend/requirements.txt
   ```

3. **Install frontend dependencies:**
   ```bash
   cd app/frontend
   npm install
   cd ../..
   ```

4. **Configure environment variables:**
   
   After deploying with `azd up`, the `.env` file is automatically created at `app/backend/.env`.
   
   If running locally without azd deployment, create `app/backend/.env` with:
   ```
   AZURE_TENANT_ID=<your-tenant-id>
   AZURE_OPENAI_ENDPOINT=https://<your-openai-service>.openai.azure.com
   AZURE_OPENAI_REALTIME_DEPLOYMENT=gpt-4o-realtime-preview
   AZURE_OPENAI_REALTIME_VOICE_CHOICE=alloy
   AZURE_SEARCH_ENDPOINT=https://<your-search-service>.search.windows.net
   AZURE_SEARCH_INDEX=<your-index-name>
   AZURE_SEARCH_SEMANTIC_CONFIGURATION=default
   AZURE_SEARCH_IDENTIFIER_FIELD=chunk_id
   AZURE_SEARCH_CONTENT_FIELD=chunk
   AZURE_SEARCH_TITLE_FIELD=title
   AZURE_SEARCH_EMBEDDING_FIELD=text_vector
   AZURE_SEARCH_USE_VECTOR_QUERY=true
   ```

   To use Entra ID authentication (recommended), omit API keys. The app will use:
   - `AzureDeveloperCliCredential` when running locally with azd
   - `DefaultAzureCredential` otherwise
   - Managed Identity when deployed to Azure

5. **Start the development server:**
   

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Azure-Samples/aisearch-openai-rag-audio](https://github.com/Azure-Samples/aisearch-openai-rag-audio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
