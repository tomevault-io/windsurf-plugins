---
trigger: always_on
description: This is a living document describing the general structure of this repo and best practices, targeted specifically at coding agents. It should be actively kept up to date if any information is discovered to be out of date, or new features are added which would be relevant to have as context in the future.
---

# Gemini Workspace Context

This is a living document describing the general structure of this repo and best practices, targeted specifically at coding agents. It should be actively kept up to date if any information is discovered to be out of date, or new features are added which would be relevant to have as context in the future.

## Project Overview

This repository contains a set of Dart packages for building AI-powered tools and applications. The core of the repository is the implementation of the Model Context Protocol (MCP), which allows AI models to interact with local development tools.

The two main packages are:

- **`dart_mcp`**: A Dart package for creating MCP (Model Context Protocol) servers and clients. It provides the foundational building blocks for communication between AI models and development tools.
- **`dart_mcp_server`**: An MCP server for Dart projects that exposes a variety of Dart and Flutter development tools to AI-assistant clients. This allows AI models to perform actions like running tests, formatting code, analyzing projects, and more.

The project is experimental and under active development.

## Code Structure

### `dart_mcp` package

This package provides the core infrastructure for MCP communication.

- **`lib/client.dart` & `lib/server.dart`**: These are the main entry points for creating MCP clients and servers. They export the `MCPClient` and `MCPServer` base classes.
- **`lib/stdio.dart`**: Provides a utility for creating a `StreamChannel` over standard I/O, which is the primary transport mechanism used for communication between the client and server.
- **`lib/src/api/api.dart`**: This is a crucial file that defines the entire MCP API as a set of Dart classes. It includes definitions for all requests, responses, and notifications, structured according to the MCP specification. This file is the source of truth for the data structures that are exchanged between the client and server.
- **`lib/src/client/client.dart`**: Contains the implementation of the `MCPClient`. This class is responsible for connecting to a server, sending requests, and handling responses and notifications. It can be extended with mixins to add support for client-side capabilities.
- **`lib/src/server/server.dart`**: Contains the implementation of the `MCPServer`. This is the base class that developers can extend to create their own MCP servers. It handles the initial handshake with the client and provides a framework for adding server-side capabilities through mixins.
- **`lib/src/shared.dart`**: Contains the `MCPBase` class, which encapsulates the common logic for JSON-RPC 2.0 communication, including registering method handlers, sending requests, and handling progress notifications.

#### Core Mixins (`dart_mcp`)

The `dart_mcp` package provides a set of mixins that add specific MCP capabilities to a client or server.

##### Client-Side Mixins (for `MCPClient`)

These are located in `lib/src/client/` and are mixed into an `MCPClient` implementation.

- **`RootsSupport`**: Manages the list of project root directories. It handles `listRoots` requests from the server and notifies the server whenever the list of roots changes. This is crucial for providing context to the server's tools.
- **`SamplingSupport`**: Allows the server to send prompts _to_ the client's LLM. The client is responsible for implementing `handleCreateMessage`, which should typically involve getting user consent before sending the prompt to the LLM and returning the response.
- **`ElicitationSupport`**: Handles requests from the server that require direct user input (e.g., asking a question, requesting a file path). The client must implement the `handleElicitation` method to define how it collects this input from the user.

##### Server-Side Mixins (for `MCPServer`)

These are located in `lib/src/server/` and are mixed into an `MCPServer` implementation.

- **`ToolsSupport`**: The core of most MCP servers. It allows you to `registerTool` and `unregisterTool`, exposing functions that the client's AI can call. It handles listing the available tools and invoking them when requested.
- **`ResourcesSupport`**: Manages exposing data and files to the client as "resources". You can `addResource`, `updateResource`, and `removeResource`. It also supports `ResourceTemplate`s for dynamic resources and handles client subscriptions to be notified of resource changes.
- **`PromptsSupport`**: Allows the server to provide a list of pre-defined prompts to the client. The client can then request a specific prompt by name, and the server will return the fully-formed prompt content.
- **`LoggingSupport`**: Provides a structured way for the server to send log messages to the client. It supports different logging levels, and the client can change the active level.
- **`CompletionsSupport`**: Adds the ability for the server to provide custom code-completion suggestions to the client.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dart-lang/ai](https://github.com/dart-lang/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
