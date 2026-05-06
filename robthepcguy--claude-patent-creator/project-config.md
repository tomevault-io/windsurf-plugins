---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

### As a Claude Code Plugin (Recommended)

```bash
# In Claude Code — add the marketplace and install the plugin
/plugin marketplace add RobThePCGuy/Claude-Patent-Creator
/plugin install claude-patent-creator-standalone@claude-patent-creator
```

### As a pip Package (MCP Server Mode)

```bash
# One-line install (works with or without venv)
pip install git+https://github.com/RobThePCGuy/Claude-Patent-Creator.git && patent-creator setup

# Restart Claude Code

# Test the system
# Ask Claude: "Search MPEP for claim definiteness requirements"
# Ask Claude: "Search for patents about neural networks filed in 2024"
```

**What happens automatically:**
1. Installs package from GitHub
2. Detects GPU (NVIDIA/Apple Silicon/CPU)
3. Uninstalls CPU PyTorch if GPU detected
4. Installs correct PyTorch (CUDA 12.8/MPS/CPU)
5. Restarts setup in subprocess with GPU-enabled PyTorch
6. Downloads MPEP PDFs and builds index with GPU acceleration
7. Registers MCP server with Claude Code

**Optional: Use venv if preferred**
```bash
python -m venv venv
venv\Scripts\activate  # Windows
source venv/bin/activate  # Linux/macOS
pip install git+https://github.com/RobThePCGuy/Claude-Patent-Creator.git && patent-creator setup
```

**What you can do now:**
- Search 100M+ patents via BigQuery (worldwide)
- Search US patent law (MPEP, 35 USC, 37 CFR)
- Search EPO patent law (EPC, EPO Guidelines) and PCT rules
- Review patent applications for USPTO, EPO, or PCT compliance
- Search EP patents via EPO OPS API (full-text claims/description)
- Generate patent-style technical diagrams
- Create complete patent applications from scratch

---

## Project Overview

**Claude Patent Creator** - An MCP server providing USPTO MPEP-based patent creation guidance using RAG (Retrieval Augmented Generation).

### Core Capabilities

| Feature | Description | Status |
|---------|-------------|--------|
| **MPEP Search** | Search Manual of Patent Examining Procedure + 35 USC + 37 CFR | Ready |
| **Patent Law Search** | Cross-jurisdiction search across US, EPO, and PCT law | Ready |
| **Patent Search** | Search 100M+ worldwide patents via BigQuery | Ready |
| **EPO Patent Search** | Search EP patents via EPO OPS API (full-text claims) | Ready |
| **IPC Search** | Search patents by IPC classification code | Ready |
| **Patent Family Search** | Find related patents across jurisdictions | Ready |
| **US Claims Review** | Automated 35 USC 112(b) compliance checking | Ready |
| **EPO Claims Review** | Automated Art. 84 EPC compliance checking | Ready |
| **US Specification Review** | Written description, enablement, best mode analysis | Ready |
| **EPO Specification Review** | Art. 83 EPC sufficiency of disclosure analysis | Ready |
| **US Formalities Check** | MPEP 608 compliance (abstract, title, drawings) | Ready |
| **EPO Formalities Check** | Rules 42-49 EPC compliance | Ready |
| **PCT Formalities Check** | PCT Rules 5-12 compliance | Ready |
| **Diagram Generation** | Patent-style technical diagrams (Graphviz) | Ready |
| **Patent Creation** | Complete patent application drafting workflow | Ready |

### Technology Stack

```
FastMCP (MCP Server Framework)
+- RAG Pipeline: FAISS + BM25 + HyDE + Cross-Encoder Reranking
+- Embeddings: BGE-base-en-v1.5 (768-dim)
+- Reranker: MS-MARCO MiniLM-L-6-v2
+- Patent Search: Google BigQuery (100M+ patents worldwide)
+- EPO Search: EPO OPS API v3.2 (EP full-text claims/description)
+- Legal Sources: MPEP + 35 USC + 37 CFR + EPC + EPO Guidelines + PCT Rules
+- Validation: Pydantic v2 with type safety
+- Logging: Structured JSON/human formats
+- GPU Acceleration: PyTorch CUDA 12.8
```

---

## Skills System

Claude will automatically activate specialized skills based on your task. These skills provide deep expertise for specific workflows:

| Skill | Activate When | What It Provides |
|-------|---------------|------------------|
| **setup-assistant** | Installing, configuring, authenticating, or troubleshooting setup | Complete installation lifecycle from pre-checks to first-use validation |
| **development-assistant** | Adding features, creating MCP tools, extending functionality | Feature development lifecycle with templates and best practices |
| **index-manager** | Building, rebuilding, or optimizing the MPEP index | MPEP index lifecycle from PDF downloads to production optimization |
| **troubleshooting-assistant** | Encountering errors, performance issues, or unexpected behavior | Systematic 6-step diagnostic methodology for all components |
| **testing-assistant** | Running tests, validation, or quality assurance | Complete test suite execution and validation workflows |
| **patent-reviewer** | Reviewing patent applications for USPTO compliance | Expert review system with automated compliance checking |
| **patent-claims-analyzer** | Reviewing claims specifically for 35 USC 112(b) | Deep-dive claims analysis (definiteness, antecedent basis, structure) |
| **patent-search** | Searching patents, prior art, or competitive intelligence | BigQuery (100M+) and PatentsView API search workflows |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RobThePCGuy/Claude-Patent-Creator](https://github.com/RobThePCGuy/Claude-Patent-Creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
