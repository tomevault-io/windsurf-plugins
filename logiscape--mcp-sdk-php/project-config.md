---
trigger: always_on
description: This is a PHP implementation of the Model Context Protocol (MCP), allowing applications to provide context for LLMs in a standardized way. The SDK implements both MCP clients and servers with support for stdio and HTTP transports.
---

# AGENTS.md

## Project Overview

This is a PHP implementation of the Model Context Protocol (MCP), allowing applications to provide context for LLMs in a standardized way. The SDK implements both MCP clients and servers with support for stdio and HTTP transports.

**Key characteristics:**
- Designed for native PHP with easy installation via Composer
- Targets PHP 8.1+ with type safety (strict_types=1)
- Supports both traditional CLI/stdio and web hosting environments
- Includes McpServer convenience wrapper for building fully functional MCP servers with just a few lines of PHP code

## Contributor-facing documentation

For non-trivial work, please also consult the governance and process docs at
the repository root: [CONTRIBUTING.md](CONTRIBUTING.md) (coding standards,
test stack, versioning policy), [ROADMAP.md](ROADMAP.md) (direction and tier
self-assessment), [SECURITY.md](SECURITY.md) (vulnerability reporting),
[GOVERNANCE.md](GOVERNANCE.md), and the deeper guides under `docs/` —
[docs/testing.md](docs/testing.md), [docs/compatibility.md](docs/compatibility.md)
(the cPanel/Apache compatibility rules), [docs/dependency-policy.md](docs/dependency-policy.md),
and [conformance/README.md](conformance/README.md) (including the
no-shortcuts-for-conformance rule).

## Development Testing Commands

### Testing Suite Installation & Dependencies
```bash
# Install dependencies
composer install

# Update dependencies
composer update

# Install pinned conformance tool version
npm install

# Install optional logging support (required for webclient and some examples)
composer require monolog/monolog
```

### Unit Tests
```bash
# Run all tests
./vendor/bin/phpunit
# or via composer script
composer test

# Run specific test file
./vendor/bin/phpunit tests/Server/ServerSessionTest.php

# Run specific test method
./vendor/bin/phpunit --filter testMethodName tests/Server/ServerSessionTest.php
```

### Static Analysis
PHPStan is configured via `phpstan.neon` and available as a dev dependency.
```bash
# Run static analysis
./vendor/bin/phpstan analyse
# or via composer script
composer analyse
```

### Regression Check
The `check` composer script runs the full regression suite (PHPUnit tests followed by PHPStan analysis). Use this before committing changes.
```bash
composer check
```

### Conformance Testing
The SDK integrates the official [MCP conformance test suite](https://github.com/modelcontextprotocol/conformance) which validates protocol compliance against the spec. The conformance tool version is pinned in `package.json` so tests are reproducible — the baseline file is tied to the installed version.

```bash
# Run all conformance tests (server + client)
composer conformance

# Run server conformance tests only
composer conformance-server

# Run client conformance tests only
composer conformance-client

# Run a single scenario
php conformance/run-conformance.php server tools-list
```

**How it works:**
- Server tests: The runner starts `conformance/everything-server.php` via PHP's built-in server, runs the conformance suite against it, then stops the server automatically via shutdown handler.
- Client tests: The conformance framework spawns `conformance/everything-client.php` with test scenario env vars and a test server URL.
- Known failures are tracked in `conformance/conformance-baseline.yml` with root cause documentation. The conformance tool uses this baseline to distinguish regressions from known limitations — if a previously passing test starts failing, it's flagged as a regression (exit code 1).

**When to run:** Run `composer conformance` after making changes to protocol handling, transport layers, session management, or McpServer. It is not included in `composer check` because it requires Node.js, but should be run separately before merging significant SDK changes.

## Building An MCP Server

The easiest and recommended way to create a new MCP server is to use the McpServer convenience wrapper. Here is a complete fully functional example that can be used as both a local MCP server or a remote MCP server.

```php
<?php
require 'vendor/autoload.php';
use Mcp\Server\McpServer;
$server = new McpServer('example-mcp-server');
$server
    ->tool('add', 'Add numbers', fn(float $a, float $b) => "Sum: " . ($a + $b))
    ->prompt('greet', 'Greeting', fn(string $name) => "Hello, {$name}!")
    ->resource(uri: 'info://php', name: 'PHP Info', callback: fn() => PHP_VERSION)
    ->run();
```

When using the convenience wrapper, `run()` is a router that uses the stdio transport on cli applications and the HTTP transport on web servers. `run()` can be replaced with `runStdio()` to force the stdio transport, or `runHttp()` to force the HTTP transport.

## Architecture Overview

### Core Component Layers

1. **Session Layer** (`Shared/BaseSession.php`)
   - Abstract base for all MCP sessions (client and server)
   - Manages JSON-RPC message routing and handler registration
   - Handles request/response matching via request IDs
   - Maintains initialization state and protocol version negotiation

2. **Client Architecture** (`Client/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [logiscape/mcp-sdk-php](https://github.com/logiscape/mcp-sdk-php) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
