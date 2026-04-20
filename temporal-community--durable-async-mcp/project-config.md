---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an invoice processing demo that integrates Temporal workflows with the Model Context Protocol (MCP). It uses MCP Tasks (SEP-1686) for async long-running operations and MCP Elicitation for human-in-the-loop approval flows.

The repo contains a shared business service (`bizservice/`) and multiple MCP server implementations:
- `async_mcp/` — Uses MCP Tasks + Elicitation with custom Temporal-backed task handlers
- `durable_sync_mcp/` — Synchronous tools (no MCP Tasks), designed for Claude Desktop over stdio

## Repository Structure

```
bizservice/              Temporal workflows, activities, worker, CLI
async_mcp/               MCP server using Tasks + Elicitation
  server.py              FastMCP server with task-enabled process_invoice tool
  temporal_task_handlers.py  Custom task handlers replacing Docket/Redis
  mcp_client/            CLI client for the async MCP server
  client_config.json     MCP client config (Claude Desktop format)
  boot-demo.sh           tmux helper to start server + worker
  tests/                 Tests for task handlers and client
durable_sync_mcp/        MCP server using synchronous tools (no Tasks)
  server.py              FastMCP server with individual tools for Claude Desktop
  claude_desktop_config.json  Sample Claude Desktop config
  README.md              Setup and usage instructions
samples/                 Sample invoice JSON files
docs/                    Design docs, research, and plans
```

## Commands

### Setup
```bash
uv venv && source .venv/bin/activate
uv pip install -e .
```

### Running the Demo
```bash
# Terminal 1: Start Temporal server
temporal server start-dev

# Terminal 2: Start the worker
python -m bizservice.worker [--fail-validate] [--fail-payment]

# Terminal 3: Start the MCP CLI client
python -m async_mcp.mcp_client [--config async_mcp/client_config.json] [--model gpt-4o]

# Or use the helper script (requires tmux)
./async_mcp/boot-demo.sh
```

### Running Tests
```bash
uv run pytest async_mcp/tests/
uv run pytest async_mcp/tests/test_task_handlers.py::TestClassName::test_name  # single test
```

### Linting/Formatting
```bash
black .
isort .
flake8
mypy .
```

## Architecture

### Workflows (`bizservice/workflows.py`)
- **InvoiceWorkflow** — Main orchestrator. Validates invoice, waits for approval signal (up to 5 days), processes line items in parallel via child workflows. Status: INITIALIZING -> PENDING-VALIDATION -> PENDING-APPROVAL -> APPROVED/REJECTED -> PAYING -> PAID/FAILED. Signals: `ApproveInvoice`, `RejectInvoice`. Queries: `GetInvoiceStatus`, `GetInvoiceData`.
- **PayLineItem** — Child workflow that waits until due date, then calls payment gateway with retry policy (3 attempts, non-retryable for INSUFFICIENT_FUNDS). Signal: `ForcePayment` skips the due-date timer and pays immediately.

### Activities (`bizservice/activities.py`)
- `validate_against_erp` — Validates invoice (30% random failure, or forced via `FAIL_VALIDATE=true`, disabled via `NO_FAIL_VALIDATE=true`)
- `payment_gateway` — Processes payment (10% INSUFFICIENT_FUNDS, 30% retryable failure, or forced via `FAIL_PAYMENT=true`, disabled via `NO_FAIL_PAYMENT=true`)

### Worker (`bizservice/worker.py`)
Connects to Temporal and polls `invoice-task-queue` for both workflows and activities. Supports `--fail-validate` and `--fail-payment` flags to force failures for testing.

### Async MCP Server (`async_mcp/server.py`)

**Tools:**
- `process_invoice` (task-enabled) — Starts a Temporal workflow and returns immediately with a task ID. The client polls `tasks/get` for status; calls `tasks/result` when `input_required` (to trigger elicitation) or `completed`/`failed`.

**Task Lifecycle:**
```
Client                          MCP Server                    Temporal
  |                               |                            |
  |-- tools/call (task meta) ---->|                            |
  |  (process_invoice)            |-- start_workflow --------->|
  |<-- CallToolResult ------------|  (taskId = workflow_id)    |
  |   (taskId, status:working)    |                            |
  |                               |                            |
  |-- tasks/get(taskId) --------->|-- query GetInvoiceStatus ->|
  |<-- status:working ------------|<-- PENDING-VALIDATION -----|
  |                               |                            |
  |-- tasks/get(taskId) --------->|-- query GetInvoiceStatus ->|
  |<-- status:input_required -----|<-- PENDING-APPROVAL -------|
  |                               |                            |
  |-- tasks/result(taskId) ------>|                            |
  |<-- elicitation: approve? -----|  (ctx.elicit within        |
  |-- user responds: approve ---->|   tasks/result handler)    |
  |                               |-- signal ApproveInvoice -->|
  |                               |-- handle.result() -------->|
  |                               |<-- "PAID" -----------------|
  |<-- CallToolResult ------------|                            |
  |   (status: PAID)              |                            |
```

### Sync MCP Server (`durable_sync_mcp/server.py`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/temporal-community) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
