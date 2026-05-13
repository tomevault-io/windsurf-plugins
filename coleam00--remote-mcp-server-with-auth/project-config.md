---
trigger: always_on
description: This guide provides implementation patterns and standards for building MCP (Model Context Protocol) servers with GitHub OAuth authentication using Node.js, TypeScript, and Cloudflare Workers. For WHAT to build, see the PRP (Product Requirement Prompt) documents.
---

# MCP Server with GitHub OAuth - Implementation Guide

This guide provides implementation patterns and standards for building MCP (Model Context Protocol) servers with GitHub OAuth authentication using Node.js, TypeScript, and Cloudflare Workers. For WHAT to build, see the PRP (Product Requirement Prompt) documents.

## Core Principles

**IMPORTANT: You MUST follow these principles in all code changes and PRP generations:**

### KISS (Keep It Simple, Stupid)

- Simplicity should be a key goal in design
- Choose straightforward solutions over complex ones whenever possible
- Simple solutions are easier to understand, maintain, and debug

### YAGNI (You Aren't Gonna Need It)

- Avoid building functionality on speculation
- Implement features only when they are needed, not when you anticipate they might be useful in the future

### Open/Closed Principle

- Software entities should be open for extension but closed for modification
- Design systems so that new functionality can be added with minimal changes to existing code

## Package Management & Tooling

**CRITICAL: This project uses npm for Node.js package management and Wrangler CLI for Cloudflare Workers development.**

### Essential npm Commands

```bash
# Install dependencies from package.json
npm install

# Add a dependency
npm install package-name

# Add a development dependency
npm install --save-dev package-name

# Remove a package
npm uninstall package-name

# Update dependencies
npm update

# Run scripts defined in package.json
npm run dev
npm run deploy
npm run type-check
```

### Essential Wrangler CLI Commands

**CRITICAL: Use Wrangler CLI for all Cloudflare Workers development, testing, and deployment.**

```bash
# Authentication
wrangler login          # Login to Cloudflare account
wrangler logout         # Logout from Cloudflare
wrangler whoami         # Check current user

# Development & Testing
wrangler dev           # Start local development server (default port 8787)

# Deployment
wrangler deploy        # Deploy Worker to Cloudflare
wrangler deploy --dry-run  # Test deployment without actually deploying

# Configuration & Types
wrangler types         # Generate TypeScript types from Worker configuration
```

## Project Architecture

**IMPORTANT: This is a Cloudflare Workers MCP server with GitHub OAuth authentication for secure database access.**

### Current Project Structure

```
/
├── src/                          # TypeScript source code
│   ├── index.ts                  # Main MCP server (standard)
│   ├── index_sentry.ts          # Sentry-enabled MCP server
│   ├── simple-math.ts           # Basic MCP example (no auth)
│   ├── github-handler.ts        # GitHub OAuth flow implementation
│   ├── database.ts              # PostgreSQL connection & utilities
│   ├── utils.ts                 # OAuth helper functions
│   └── workers-oauth-utils.ts   # Cookie-based approval system
├── PRPs/                        # Product Requirement Prompts
│   ├── README.md
│   └── templates/
│       └── prp_base.md
├── wrangler.jsonc              # Main Cloudflare Workers configuration
├── wrangler-simple.jsonc       # Simple math example configuration
├── package.json                # npm dependencies & scripts
├── tsconfig.json               # TypeScript configuration
├── worker-configuration.d.ts   # Generated Cloudflare types
└── CLAUDE.md                   # This implementation guide
```

### Key File Purposes (ALWAYS ADD NEW FILES HERE)

**Main Implementation Files:**

- `src/index.ts` - Production MCP server with GitHub OAuth + PostgreSQL
- `src/index_sentry.ts` - Same as above with Sentry monitoring integration
- `src/simple-math.ts` - Basic MCP server example (calculator without auth)

**Authentication & Security:**

- `src/github-handler.ts` - Complete GitHub OAuth 2.0 flow
- `src/workers-oauth-utils.ts` - HMAC-signed cookie approval system
- `src/utils.ts` - OAuth token exchange and URL construction helpers

**Database Integration:**

- `src/database.ts` - PostgreSQL connection pooling, SQL validation, security

**Configuration Files:**

- `wrangler.jsonc` - Main Worker config with Durable Objects, KV, AI bindings
- `wrangler-simple.jsonc` - Simple example configuration
- `tsconfig.json` - TypeScript compiler settings for Cloudflare Workers

## Development Commands

### Core Workflow Commands

```bash
# Setup & Dependencies
npm install                  # Install all dependencies
npm install --save-dev @types/package  # Add dev dependency with types

# Development
wrangler dev                # Start local development server
npm run dev                 # Alternative via npm script

# Type Checking & Validation
npm run type-check          # Run TypeScript compiler check
wrangler types              # Generate Cloudflare Worker types
npx tsc --noEmit           # Type check without compiling

# Testing
npx vitest                  # Run unit tests (if configured)

# Code Quality
npx prettier --write .      # Format code
npx eslint src/            # Lint TypeScript code
```

### Environment Configuration

**Environment Variables Setup:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coleam00/remote-mcp-server-with-auth](https://github.com/coleam00/remote-mcp-server-with-auth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
