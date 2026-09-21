---
trigger: always_on
description: > **Official docs**: [https://docs.pinecone.io/](https://docs.pinecone.io/) - For complete API reference, advanced features, and detailed guides.
---

# null

# Pinecone quick reference for agents

> **Official docs**: [https://docs.pinecone.io/](https://docs.pinecone.io/) - For complete API reference, advanced features, and detailed guides.

This guide covers critical gotchas, best practices, and common patterns specific to this project. For anything not covered here, consult the official Pinecone documentation.

***

## ⚠️ Critical: Installation & SDK

**ALWAYS use the current SDK:**

```bash  theme={null}
pip install pinecone          # ✅ Correct (current SDK)
pip install pinecone-client   # ❌ WRONG (deprecated, old API)
```

**Current API (2025):**

```python  theme={null}
from pinecone import Pinecone  # ✅ Correct import
```

***

## 🔧 CLI vs SDK: When to Use Which

**Use the Pinecone CLI for:**

* ✅ **Creating indexes** - `pc index create`
* ✅ **Deleting indexes** - `pc index delete`
* ✅ **Configuring indexes** - `pc index configure` (replicas, deletion protection)
* ✅ **Listing indexes** - `pc index list`
* ✅ **Describing indexes** - `pc index describe`
* ✅ **Creating API keys** - `pc api-key create`
* ✅ **One-off inspection** - Checking stats, configuration
* ✅ **Development setup** - All initial infrastructure setup

**Use the Python SDK for:**

* ✅ **Data operations in application code** - upsert, query, search, delete RECORDS
* ✅ **Runtime checks** - `pc.has_index()`, `index.describe_index_stats()`
* ✅ **Automated workflows** - Any data operations that run repeatedly
* ✅ **Production data access** - Reading and writing vectors/records

**❌ NEVER use SDK for:**

* Creating, deleting, or configuring indexes in application code
* One-time administrative tasks

### Installing the Pinecone CLI

**macOS (Homebrew):**

```bash  theme={null}
brew tap pinecone-io/tap
brew install pinecone-io/tap/pinecone

# Upgrade later
brew update && brew upgrade pinecone
```

**Other platforms:**
Download from [GitHub Releases](https://github.com/pinecone-io/cli/releases) (Linux, Windows, macOS)

### CLI Authentication

Choose one method:

**Option 1: User login (recommended for development)**

```bash  theme={null}
pc login
pc target -o "my-org" -p "my-project"
```

**Option 2: API key**

```bash  theme={null}
export PINECONE_API_KEY="your-api-key"
# Or: pc auth configure --global-api-key <api-key>
```

**Option 3: Service account**

```bash  theme={null}
export PINECONE_CLIENT_ID="your-client-id"
export PINECONE_CLIENT_SECRET="your-client-secret"
```

**Full CLI reference:** [https://docs.pinecone.io/reference/cli/command-reference](https://docs.pinecone.io/reference/cli/command-reference)

***

## Quickstarts

> **Important for all quickstarts**: Execute all steps completely. Keep setup minimal (directories, venv, dependencies only). Do not expect the user to satisfy any prerequisites except providing API keys. For summaries, use only README.md and SUMMARY.md.

When you are asked to help get started with Pinecone, ask the user to choose an option:

* Quick test: Create an index, upsert data, and perform semantic search.

* Choose a use case:
  * Search: Build a semantic search system that returns ranked results from your knowledge base. This pattern is ideal for search interfaces where users need a list of relevant documents with confidence scores.

  * RAG: Build a multi-tenant RAG (Retrieval-Augmented Generation) system that retrieves relevant context per tenant and feeds it to an LLM to generate answers. Each tenant (organization, workspace, or user) has isolated data stored in separate Pinecone namespaces. This pattern is ideal for knowledge bases, customer support platforms, and collaborative workspaces.

  * Recommendations: Build a recommendation engine that suggests similar items based on semantic similarity. This pattern is ideal for e-commerce, content platforms, and user personalization systems.

Based on the choice, use the appropriate pattern.

### Setup Prerequisites (all quickstarts)

Before starting any quickstart, complete these steps:

1. **Set up Python environment**: Create project directory, virtual environment, and install Pinecone SDK
2. **Install CLI**: Run `pc version` to check. If not installed: `brew tap pinecone-io/tap && brew install pinecone-io/tap/pinecone` (macOS) or download from [GitHub releases](https://github.com/pinecone-io/cli/releases). If already installed, upgrade: `brew update && brew upgrade pinecone`
3. **Configure API key**: Ask user for Pinecone API key, set as `PINECONE_API_KEY` env variable, then run `pc auth configure --api-key $PINECONE_API_KEY`
4. **For RAG quickstart only**: Also obtain and set `OPENAI_API_KEY` or `ANTHROPIC_API_KEY`

### Quick test

Complete [Setup Prerequisites](#setup-prerequisites-all-quickstarts) first.

**Step 1. Implement semantic search**

1. Create an index called "agentic-quickstart-test" with an integrated embedding model that can handle text documents. Use the Pinecone CLI for this. Use the API key env variable to authenticate.

   ```bash  theme={null}
   pc index create -n agentic-quickstart-test -m cosine -c aws -r us-east-1 --model llama-text-embed-v2 --field_map text=content
   ```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jack-MillionMedia/Medcan-Agent](https://github.com/Jack-MillionMedia/Medcan-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
