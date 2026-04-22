---
trigger: always_on
description: This repository is a personal knowledge base and code laboratory for Machine Learning and Generative AI studies. It contains documentation (MkDocs), Jupyter notebooks, and Python code samples covering classical ML, deep learning, and LLM/agent frameworks.
---

# ML-Studies Project Context

This repository is a personal knowledge base and code laboratory for Machine Learning and Generative AI studies. It contains documentation (MkDocs), Jupyter notebooks, and Python code samples covering classical ML, deep learning, and LLM/agent frameworks.

## Project Structure

```
ML-studies/
├── src/                     # Shared library code
│   └── mlstudies/           # Reusable utilities
│       ├── config.py        # Centralized env vars and settings
│       ├── rag.py           # RAG utilities (text splitting, vector stores)
│       └── llm.py           # LLM client factories
├── examples/                # Python implementations organized by technology
│   ├── llm-langchain/       # LangChain/LangGraph examples with multiple LLM providers
│   ├── llm-ollama/          # Local LLM experiments with Ollama
│   ├── llamaindex/          # LlamaIndex RAG implementations
│   ├── pytorch/             # PyTorch deep learning experiments
│   ├── ml-python/           # Classical ML with scikit-learn
│   ├── deep-neural-net/     # Neural network fundamentals
│   └── haystack/            # Haystack framework experiments
├── docs/                    # MkDocs markdown documentation
├── notebooks/               # Jupyter notebooks for interactive learning
├── e2e-demos/               # End-to-end application demos
├── techno/                  # Technology-specific experiments (Gradio, Streamlit, etc.)
└── data/                    # Sample datasets for experiments
```

## Technology Stack

### Core Dependencies
- Python 3.11+
- Package management: **uv** with `pyproject.toml` per subproject

### Machine Learning
- scikit-learn: Classical ML algorithms (classifiers, regression, clustering)
- PyTorch: Deep learning, distributed training (DDP)
- TensorFlow/Keras: Neural networks (legacy examples)

### LLM Frameworks
- LangChain: Chains, prompts, document loaders, embeddings
- LangGraph: Stateful agent graphs with tool calling
- LlamaIndex: RAG pipelines and query engines

### LLM Providers
- OpenAI: GPT models via langchain-openai
- Anthropic: Claude models
- AWS Bedrock: Claude, Stability Diffusion
- IBM WatsonX: Granite, Llama models
- Mistral: via langchain-mistralai
- Ollama: Local model inference
- Cohere: Embeddings and chat
- Google Gemini

### Vector Stores
- ChromaDB: Primary vector store for RAG
- FAISS: Fast similarity search
- OpenSearch: Enterprise search

### UI Frameworks
- Gradio: Quick ML demos with UI
- Streamlit: Data apps and dashboards
- TaiPy: Enterprise data pipelines with UI
- NiceGUI: Python-native web UIs

### Documentation
- MkDocs with Material theme
- Jupyter notebooks for interactive content

## Coding Conventions

### File Organization
- Each subproject has its own `pyproject.toml` managed by **uv**
- Environment variables stored in `.env` at project root
- Use `dotenv` for loading environment configuration

### Python Style

**Preferred**: Use the shared config module:
```python
from mlstudies.config import OPENAI_API_KEY, ANTHROPIC_API_KEY

# Or import specific utilities
from mlstudies.llm import get_openai_chat, get_anthropic_chat
from mlstudies.rag import create_chroma_vectorstore, split_documents
```

**Legacy pattern** (still found in some files):
```python
from dotenv import load_dotenv
import os

load_dotenv("../../.env")
api_key = os.getenv("OPENAI_API_KEY")
```

### LangChain/LangGraph Patterns
```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages

# Define state with TypedDict
class State(TypedDict):
    messages: Annotated[list, add_messages]

# Build graph with nodes and edges
graph_builder = StateGraph(State)
graph_builder.add_node("node_name", node_function)
graph_builder.set_entry_point("node_name")
graph = graph_builder.compile()
```

### RAG Implementation Pattern
```python
from langchain_community.document_loaders import WebBaseLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_openai import OpenAIEmbeddings

# 1. Load documents
loader = WebBaseLoader(web_paths=(url,))
docs = loader.load()

# 2. Split into chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = splitter.split_documents(docs)

# 3. Create vector store with embeddings
vectorstore = Chroma.from_documents(documents=splits, embedding=OpenAIEmbeddings())

# 4. Create retriever
retriever = vectorstore.as_retriever(search_type="similarity", search_kwargs={"k": 6})
```

### Agent Pattern with Tools
```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain.tools.retriever import create_retriever_tool

# Create tools
retriever_tool = create_retriever_tool(retriever, "tool_name", "Tool description")
tools = [retriever_tool, other_tools]

# Create agent with prompt
agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# Invoke

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jbcodeforce) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
