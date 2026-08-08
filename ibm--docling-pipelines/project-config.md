---
trigger: always_on
description: The orchestrator mode is a strategic workflow coordinator designed to handle complex, multi-faceted tasks by intelligently breaking them down into manageable subtasks and delegating them to specialized modes. It acts as a high-level project manager, ensuring efficient task execution through proper mode selection and coordination.
---

# Orchestrator Mode Agent

## Overview
The orchestrator mode is a strategic workflow coordinator designed to handle complex, multi-faceted tasks by intelligently breaking them down into manageable subtasks and delegating them to specialized modes. It acts as a high-level project manager, ensuring efficient task execution through proper mode selection and coordination.

## Repository Context
The docpipe project is a modular, operator-based data processing framework designed for building flexible data pipelines. Key architectural characteristics:

- **Operator-Based Architecture**: 20+ specialized operators organized into 5 categories (Extract, Ingest, Functional, Quality, VectorDB)
- **PyArrow Data Format**: All data flows through the pipeline as PyArrow tables, ensuring efficient memory usage and interoperability
- **DAG-Based Workflow Execution**: Flows are defined as JSON configurations representing directed acyclic graphs (DAGs) of operator steps (see [`docs/guides/FLOW_AUTHORING_FORMAT.md`](docs/guides/FLOW_AUTHORING_FORMAT.md))
- **Prefect Orchestration**: The orchestrator layer uses Prefect for managing workflow execution, parallel processing, and task dependencies
- **Modern AI/ML Integrations**: Native support for Ollama (LLM operations), Docling (document processing), and OpenSearch (vector storage)
- **Multi-Provider Support**: Flexible ingest operators supporting local files, S3, CSV, and multi-provider sources

### User Guide Reference
For new user setup and complete pipeline execution instructions, refer to [`USER_GUIDE_PIPELINE_SETUP.md`](USER_GUIDE_PIPELINE_SETUP.md). This comprehensive guide covers:
- Prerequisites and installation (Python 3.12, uv, dependencies)
- Ollama setup for LLM operations and embeddings
- OpenSearch setup with Podman/Docker for vector storage
- Flow configuration structure and operator examples
- Step-by-step pipeline execution
- Verification, testing, and troubleshooting

**Note:** Consult this guide when helping users set up their environment or execute their first pipeline.

## User Entry Points

Docling Pipelines provides multiple interfaces for interacting with the framework:

### 1. CLI Entry Point
Primary interface using the `docling-pipelines` command:

```bash
# Flow execution
docling-pipelines --flow-file <path-to-flow.json>

# Flow validation
docling-pipelines --flow-file flow.json --validate

# List operators
docling-pipelines --list-operators [--verbose]

# Log level control (via environment variable)
DS_LOG_LEVEL=DEBUG docling-pipelines --flow-file flow.json
```

### 2. Python Library
Programmatic access via `DocpipeFlowManager`:

```python
from docpipe.lib.docpipe_flow_manager import DocpipeFlowManager

# Execute from file
manager = DocpipeFlowManager(flow_file="path/to/flow.json")
result = manager.execute()

# Execute from dict
manager = DocpipeFlowManager(flow_def=flow_dict)
result = manager.execute()

# Validate flow
validation_result = manager.validate()

# List operators
DocpipeFlowManager.list_operators(verbose=True)
```

### 3. REST API Service
FastAPI server for web service integration (development status):

```bash
# Start server
uvicorn docpipe.api.main:app --reload --host 0.0.0.0 --port 8000

# Interactive docs at http://localhost:8000/docs
```

**Key endpoints:** `/api/v1/flows`, `/api/v1/operators`, `/api/v1/job_runs`

**Authentication:** LDAP with JWT tokens, OAuth2/OIDC support

## Role
Strategic workflow coordinator that breaks down complex tasks and delegates to specialized modes.

## Key Capabilities
- **Task decomposition and delegation**: Analyzes complex requests and breaks them into logical, sequential subtasks
- **Workflow coordination across multiple modes**: Manages the execution flow between different specialized modes (code, advanced, architect, etc.)
- **Progress tracking and result synthesis**: Monitors subtask completion and combines results into cohesive outcomes
- **Mode selection and task routing**: Intelligently selects the most appropriate mode for each subtask based on requirements
- **Understanding JSON flow definitions**: Interprets DAG-structured flow configurations with operator nodes and dependencies
- **Knowledge of 20+ available operators**: Familiar with Extract, Ingest, Functional, Quality, and VectorDB operators
- **Flow validation and operator configuration**: Ensures proper operator parameters and data flow connections
- **Integration awareness**: Understands requirements for Ollama, Docling, and OpenSearch integrations

## Available Operators

Docling Pipelines provides 20+ operators across 5 categories:
- **Extract**: Document text and entity extraction (ExtractOperator with multiple modes)
- **Ingest**: Data source ingestion (IngestLocalOperator, IngestSourceOperator)
- **Functional**: Data transformation (Chunker, EmbeddingsOperator, BranchingOperator, NoopOperator, etc.)
- **Quality**: Data quality checks (Dedup, Redaction, LanguageDetection, SQLFilter, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/docling-pipelines](https://github.com/IBM/docling-pipelines) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
