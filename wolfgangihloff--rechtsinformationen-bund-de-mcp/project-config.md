---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Core Commands
- `npm run build` - Compile TypeScript to JavaScript in dist/
- `npm run dev` - Run development server with tsx (TypeScript execution)
- `npm start` - Run production server from dist/index.js
- `npm test` - Run golden test cases (should show 100% success rate)
- `npm run setup` - Complete setup: install, build, and test

### Testing & Debugging
- `npm run test:api` - Test API connectivity to rechtsinformationen.bund.de
- `npm run verify` - Verify complete setup
- `npm run check-config` - Show Claude Desktop config file path

### Configuration
- `npm run claude-config` - Generate correct MCP server config for Claude Desktop
- `./quick-setup.sh` - Complete automated setup script

## Architecture Overview

This is an MCP (Model Context Protocol) server that provides access to the official German Federal Legal Information Portal (rechtsinformationen.bund.de). The server acts as a bridge between Claude Desktop and the German legal database.

### Core Architecture
- **MCP Server**: Built using @modelcontextprotocol/sdk
- **Communication**: stdio transport for Claude Desktop integration
- **API Client**: Axios-based client for rechtsinformationen.bund.de REST API
- **Semantic Search**: Fuse.js for fuzzy matching and semantic search capabilities

### Key Components

#### Main Server Class (`src/index.ts`)
- `RechtsinformationenBundDeMCPServer`: Main server class handling MCP protocol
- Tool registration and request handling
- API communication with German legal database

#### Six German Legal Search Tools (Usage Priority Order)

**PRIMARY TOOL (Use First):**
1. `semantische_rechtssuche` ⭐ - **ALWAYS USE FIRST for ANY German legal question** - Intelligent semantic search with misconception correction, legal concept mapping, and comprehensive rechtsinformationen.bund.de search. Handles date filtering properly and corrects common legal misconceptions automatically.

**SECONDARY TOOLS (Use for Follow-up):**
2. `deutsche_gesetze_suchen` - Federal legislation search (WARNING: Has date filtering limitations)
3. `rechtsprechung_suchen` - Court decisions search (Use only for specific court filtering)  
4. `alle_rechtsdokumente_suchen` - Comprehensive document search (WARNING: Has date filtering limitations)
5. `gesetz_per_eli_abrufen` - Specific law retrieval by ELI (Use only when you have specific ELI identifier)
6. `dokument_details_abrufen` - Detailed document retrieval (Use only when you have specific document ID)

### API Integration Details

#### Base URL
- Development/Test: `https://testphase.rechtsinformationen.bund.de/v1`
- Endpoints: `/legislation`, `/case-law`, `/document`

#### URL Handling and Data Model Understanding

The rechtsinformationen.bund.de API follows **FRBR (Functional Requirements for Bibliographic Records)** model with three distinct levels:

**1. Work Level (Abstract Concept)**
- Represents the law as an intellectual creation
- Example: "Das Grundgesetz" as a whole, independent of versions
- URL pattern: `/v1/legislation/eli/bund/...` (without date/version)

**2. Expression Level (Concrete Version)**
- Represents a specific publication or version of the law
- Contains metadata: legislation date, publication date, effective date, in-force status
- **Critical for validity questions**: "Is this law still in force?", "When did this become effective?"
- URL pattern: `/v1/legislation/eli/bund/bgbl-1/2016/s3234/2025-01-01/1/deu`
- Access via: `workExample['@id']` in API responses
- **Returns JSON-LD with metadata**

**3. Manifestation Level (Physical/Digital Format)**
- Represents the format in which the expression is available
- JSON-LD: `/v1/legislation/eli/...` (machine-readable metadata)
- HTML: `/norms/eli/...` (human-readable web page)
- PDF, XML, etc.

**Key Implementation Details**:
- **For user-facing links**: Convert `/v1/legislation/eli/...` → `/norms/eli/...` (HTML viewer)
- **For metadata queries**: Use `/v1/legislation/eli/...` (JSON-LD) directly
- **Metadata available at search result level**:
  - `legislationDate` - When the law was passed/enacted
  - `datePublished` - When published in Federal Law Gazette (BGBl)
  - `workExample['@id']` - URL to the specific versioned document (Expression level)
- **For detailed validity questions**: Fetch the specific document using `workExample['@id']` or `gesetz_per_eli_abrufen` to access:
  - `temporalCoverage` - Date range when law is/was in force (if available)
  - `inForce` - Boolean indicating current validity status (if available)
  - Full text content and structure

**Example - SGB IX**:
- **Expression URL** (JSON-LD): `https://testphase.rechtsinformationen.bund.de/v1/legislation/eli/bund/bgbl-1/2016/s3234/2025-01-01/1/deu`
  - Contains: legislationDate, datePublished, inForce status
  - Use for: "When did SGB IX become effective?", "Is SGB IX still valid?"
- **Manifestation URL** (HTML): `https://testphase.rechtsinformationen.bund.de/norms/eli/bund/bgbl-1/2016/s3234/2025-01-01/1/deu`
  - Contains: Human-readable text of the law
  - Use for: Reading the actual law text

**Agent Guidance**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wolfgangihloff/rechtsinformationen-bund-de-mcp](https://github.com/wolfgangihloff/rechtsinformationen-bund-de-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
