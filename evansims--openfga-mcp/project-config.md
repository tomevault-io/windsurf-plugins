---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a PHP implementation of a Model Context Protocol (MCP) server for OpenFGA, providing AI agents with tools to manage and query OpenFGA authorization servers.

## Model Context Protocol Concepts

Model Context Protocol solves the fundamental problem of connecting AI to your existing tools and data in a standardized, secure way. MCP creates a common language between AI models and external systems. Instead of building point-to-point integrations, you implement MCP servers that expose your tools, data, and prompts through a standard interface. Any MCP-compatible AI can then connect to any MCP server.

The real power emerges when your AI can seamlessly move between reading your calendar (resource), sending emails (tool), querying databases (resource template), and following company-specific workflows (prompts) - all through the same protocol, with proper security boundaries.

**Tools** are executable functions that perform actions or operations. Think of them as API endpoints the AI can call - they do things. A tool might send an email, create a database record, or run a calculation. Tools typically modify state or trigger external processes.

**Resources** are read-only data sources the AI can access for information. They provide content rather than perform actions. A resource might be a file, database query result, or API response that gives the AI context or data to work with.

**Resource Templates** are parameterized blueprints for generating resources dynamically. Instead of static resources, templates let you create resources on-demand based on input parameters. For example, a template might generate a user profile resource when given a user ID, or create a report resource based on date ranges.

**Prompts** are reusable prompt templates that can be invoked with parameters to generate structured prompts for the AI. They're essentially parameterized instructions or context that help shape how the AI approaches specific tasks.

The key distinction: tools _act_, resources _inform_, templates _generate_, and prompts _guide_. Tools change the world, resources describe it, templates make resources flexible, and prompts shape how the AI thinks about both.

This separation keeps concerns clean - you're not mixing data access with actions, and you can build flexible, reusable components that compose well together.

## Development Commands

### Linting and Code Quality

- `composer lint` - Run all linters (PHPStan, Psalm, Rector, PHP-CS-Fixer)
- `composer lint:fix` - Apply available automatic fixes from linters

### Testing

- `composer test` - Run all tests using Pest framework
- `composer test:unit` - Run unit tests using Pest framework
- `composer test:integration` - Run integration tests using Pest framework inside a Docker environment

## Architecture

The project has a minimal structure focused on implementing an MCP server:

- **src/Server.php**: Main class that implements the MCP server
- **src/Helpers.php**: Helper functions for MCP server (includes `isOfflineMode()`)
- **src/OfflineClient.php**: Offline client implementation that allows server to run without OpenFGA
- **src/Tools/**: Directory containing classes that expose tools for MCP client to invoke
- **src/Resources/**: Directory containing classes that expose resources for MCP client to access
- **src/Templates/**: Directory containing classes that expose resource templates for MCP client to generate resources from
- **src/Prompts/**: Directory containing classes that expose prompts for MCP client to generate structured prompts from
- **src/Documentation/**: Directory containing documentation indexing and chunking system for SDK knowledge base
- Built on top of `php-mcp/server` framework
- Uses `evansims/openfga-php` for OpenFGA client functionality

## Operating Modes

The server supports two distinct operating modes:

### Online Mode (Full Features)
When `OPENFGA_MCP_API_URL` is configured (or authentication credentials provided), the server operates with full functionality:
- All Tools work (create/manage stores, models, relationships)
- All Resources provide live data from OpenFGA
- Dynamic Completions fetch data from the server
- Prompts work as normal

### Offline Mode (Planning & Coding Only)
When no OpenFGA configuration is provided, the server operates in offline mode:
- Tools return error messages guiding users to configure OpenFGA
- Resources return error responses with helpful messages
- Completions return static defaults (e.g., common relations, object patterns)
- **Prompts work normally** - this is the key feature of offline mode

The server automatically detects the mode based on environment variables. The `OfflineClient` class implements the `ClientInterface` to maintain architectural consistency.

### Detecting Mode
```php
// Check if in offline mode
if (isOfflineMode()) {
    // Handle offline case
}

// In base classes
$error = $this->checkOfflineMode('operation name');
if (null !== $error) {
    return $error;
}
```

## Configuration Methods

The server supports two methods for configuration:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evansims/openfga-mcp](https://github.com/evansims/openfga-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
