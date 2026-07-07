---
trigger: always_on
description: This document provides an overview of the ALOHA project.
---

# ALOHA Project Documentation

This document provides an overview of the ALOHA project.

<!-- product.md -->

# Product Overview

## Project Name

ALOHA (AI Logical Orchestrator Hub for Agents)

## Purpose

ALOHA is a centralized hub for managing and interconnecting data and AI agents. Built on the Model Context Protocol (MCP) and Agent-to-Agent (A2A) protocol, it enables seamless communication between applications, MCP servers, and AI agents, enhancing tool discoverability and interoperability across AI agent systems.

## Target Audience

- **AI/ML Developers**: Building agentic systems that need to connect to multiple tools and services
- **Enterprise Teams**: Organizations requiring centralized management of MCP infrastructure
- **Agent Developers**: Teams testing and debugging MCP server and A2A agent implementations
- **System Integrators**: Engineers deploying proxy servers for simplified agent-tool connections

## Key Features

### Multi-Protocol Agent Connectivity

- **MCP Protocol**: Connect agents to MCP servers for tool access and resource management
- **A2A Protocol**: Connect agents to other AI agents for inter-agent communication and task delegation
- Unified management interface for both protocol types
- Seamless switching between connection modes per agent

### MCP Server Management

- Connect to both local and remote MCP servers
- Support for multiple authentication strategies:
  - Basic authentication
  - Bearer token
  - OAuth2 (planned)
  - Custom plugins (extendable architecture)
- Test MCP server responses directly from the browser
- Debug and improve tool implementations

### A2A Agent Integration

- Register and manage A2A-compatible agents
- Send tasks to remote agents and receive streaming responses
- Support for A2A artifacts and task status tracking
- Real-time event streaming for agent interactions

### Proxy Servers

- Deploy proxy servers to simplify connections between agents/applications and tools
- MCP proxy for tool aggregation
- A2A proxy for agent-to-agent routing
- Single endpoint management with identity propagation
- Reduce boilerplate code by centralizing authentication and routing

### Agent Development & Testing

- Run agentic loops directly in the browser using OpenAI-compatible endpoints
- Test MCP server and A2A agent performance with real workflows
- Browser-based testing environment for rapid iteration
- Visual event streaming for debugging agent interactions

### Authentication & Identity Propagation

- Plugin-based authentication system
- OIDC integration with Keycloak
- Dynamic client registration
- Identity propagation across services
- Permission-based access control

### Database Flexibility

- MongoDB as primary database
- PostgreSQL as alternative database option
- Configurable via environment variables

## Business Objectives

- **Simplify Agent Integration**: Reduce complexity of connecting agents to MCP servers and other agents via A2A
- **Accelerate Development**: Provide out-of-the-box testing and debugging tools for both protocols
- **Enable Enterprise Adoption**: Support enterprise authentication and identity management
- **Foster Interoperability**: Standardize on MCP and A2A protocols for tool discovery and agent communication

## Success Metrics

- Number of MCP servers successfully connected
- Number of A2A agents integrated
- Agent execution success rate across both protocols
- Developer time saved in testing and debugging
- Enterprise deployments with OIDC integration

<!-- tech.md -->

# Technology Stack

## Programming Languages

- **TypeScript 5.9+**: Primary language for all packages
- **JavaScript**: Build configurations and some tooling

## Frontend Stack

### Core Framework

- **React 19.1**: UI library
- **React Router 7.8**: Client-side routing
- **Vite 7.1**: Build tool and dev server

### UI Components & Styling

- **TailwindCSS 4.1**: Utility-first CSS framework
- **Radix UI**: Headless component primitives (dialogs, dropdowns, tooltips, etc.)
- **shadcn/ui**: Pre-built accessible components
- **Lucide React**: Icon library
- **Motion (Framer Motion)**: Animation library

### State Management & Data

- **React Hook Form 7.62**: Form state management
- **Zod 4.x**: Schema validation
- **RxJS 7.8**: Reactive programming for real-time updates

### Code Display

- **Prism.js**: Syntax highlighting
- **react-simple-code-editor**: Code editing components

## Backend Stack

### Core Framework

- **Node.js**: Runtime environment
- **Express 5.1**: Web server framework
- **TypeScript (ES Modules)**: `"type": "module"` in package.json

### Database

- **MongoDB 6.19**: Primary database option
- **PostgreSQL 8.x**: Alternative database option (via pg driver)
- **connect-mongo**: Session store for Express (MongoDB)
- **connect-pg-simple**: Session store for Express (PostgreSQL)

### Authentication & Security

- **openid-client 6.8**: OpenID Connect client
- **jose 6.1**: JWT/JWE/JWS implementation
- **express-session 1.18**: Session middleware
- **cookie-parser**: Cookie handling

### MCP & Agent Integration

- **@modelcontextprotocol/sdk 1.26**: Model Context Protocol implementation
- **@a2a-js/sdk 0.3**: Agent-to-Agent protocol implementation (see [A2A Protocol Documentation](../.kiro/knowledge/a2a.md))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ec-jrc/aloha](https://github.com/ec-jrc/aloha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
