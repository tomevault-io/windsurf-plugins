---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that processes Outlook emails, generates embeddings using Ollama, and provides semantic search capabilities. It's designed to be used with Claude Desktop and complies with the **MCP 2025-06-18 specification**.

**Cross-Platform Support**: The server supports Windows, macOS, and any platform via Microsoft Graph API through a provider-based connector architecture.

## Development Setup

### Prerequisites
- Python 3.10+
- Ollama running locally with `nomic-embed-text` model
- Microsoft Outlook installed (Windows/Mac) OR Azure AD app for Graph API
- MongoDB server running locally or accessible via network

### Installation

```bash
# Install uv if needed
pip install uv

# Create and activate virtual environment
uv venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate

# Install core dependencies
uv pip install -e .

# For Windows (adds pywin32)
uv pip install -e ".[windows]"

# For Graph API support (cross-platform)
uv pip install -e ".[graph]"

# For all optional dependencies
uv pip install -e ".[all]"

# Ensure Ollama model is available
ollama pull nomic-embed-text
```

### Running the Server

**STDIO Transport (default for Claude Desktop):**
```bash
python src/mcp_server.py
```

**HTTP Transport (for testing):**
```bash
python src/mcp_server.py --http
```
Server will be available at `http://localhost:8000/mcp`

### Configuration

The server is configured via environment variables (typically set in Claude Desktop config):

**Core Configuration:**
- `MONGODB_URI`: MongoDB connection string
- `SQLITE_DB_PATH`: Path to SQLite database file
- `EMBEDDING_BASE_URL`: Ollama server URL (default: http://localhost:11434)
- `EMBEDDING_MODEL`: Embedding model name (nomic-embed-text)
- `COLLECTION_NAME`: MongoDB collection name (required)
- `PROCESS_DELETED_ITEMS`: Whether to process Deleted Items folder (default: "false")

**Platform/Provider Configuration:**
- `OUTLOOK_PROVIDER`: Provider to use - `auto`, `windows`, `mac`, or `graph` (default: "auto")
- `LOCAL_TIMEZONE`: Timezone for date conversion (default: "UTC", e.g., "America/Chicago")

**Graph API Configuration (for `graph` provider):**
- `GRAPH_CLIENT_ID`: Azure AD application (client) ID
- `GRAPH_CLIENT_SECRET`: Azure AD client secret
- `GRAPH_TENANT_ID`: Azure AD tenant ID (default: "common")
- `GRAPH_USER_EMAILS`: Mailboxes to access - comma-separated list or "All" (default: "All")
  - `"All"`: Discover and access all licensed mailboxes in tenant
  - `"email1@domain.com,email2@domain.com"`: Access specific mailboxes only
- `GRAPH_USER_EMAIL`: (Legacy) Single user email - use `GRAPH_USER_EMAILS` instead

## Architecture

### High-Level Data Flow

1. **Email Retrieval**: Platform connector retrieves emails (Windows COM, Mac AppleScript, or Graph API)
2. **Primary Storage**: Emails stored in SQLite (`SQLiteHandler`) for fast filtering and full-text search
3. **Embedding Generation**: `EmbeddingProcessor` creates embeddings via Ollama's nomic-embed-text
4. **Vector Storage**: Embeddings stored in MongoDB (`MongoDBHandler`) for semantic search
5. **MCP Interface**: `mcp_server.py` exposes tools via FastMCP framework

### Provider-Based Connector Architecture

The application uses a **provider-based abstraction** for cross-platform support:

```
src/connectors/
├── __init__.py           # Package exports
├── base.py               # OutlookConnectorBase ABC
├── mailbox_info.py       # Platform-agnostic MailboxInfo dataclass
├── factory.py            # create_connector() with auto-detection
├── windows_connector.py  # Windows COM via pywin32
├── mac_connector.py      # macOS AppleScript via osascript
└── graph_connector.py    # Microsoft Graph API (cross-platform)
```

**Provider Selection:**
- `auto` (default): Detects best available provider
  - Windows → `windows` (COM automation)
  - macOS → `mac` (AppleScript)
  - Other/fallback → `graph` (requires Azure AD setup)
- `windows`: Forces Windows COM (requires pywin32)
- `mac`: Forces macOS AppleScript (requires Outlook for Mac)
- `graph`: Forces Microsoft Graph API (works anywhere with Azure AD credentials)

### Hybrid Search Strategy

The application uses a **dual-database architecture**:

**SQLite Database (`SQLiteHandler`)**:
- Primary storage for all email metadata and content
- Full-text search on subject/body/sender
- Date range filtering, folder filtering, sender filtering
- Tracks processing status (whether embeddings have been generated)
- Fast SQL queries for structured filtering

**MongoDB (`MongoDBHandler`)**:
- Stores vector embeddings for semantic search
- Enables similarity search across email content
- Metadata stored alongside embeddings for retrieval

This design allows combining semantic search (MongoDB) with structured filtering (SQLite).

### Key Components

**`connectors/` Package** (`src/connectors/`):
- `OutlookConnectorBase`: Abstract base class defining the connector interface
- `MailboxInfo`: Platform-agnostic dataclass for mailbox information
- `WindowsOutlookConnector`: Windows COM implementation using pywin32

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cam10001110101/mcp-server-outlook-email](https://github.com/Cam10001110101/mcp-server-outlook-email) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
