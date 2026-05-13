---
trigger: always_on
description: This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides comprehensive guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Development Environment](#development-environment)
5. [Build System](#build-system)
6. [Testing Framework](#testing-framework)
7. [Security Model](#security-model)
8. [API Documentation](#api-documentation)
9. [Deployment](#deployment)
10. [Configuration](#configuration)
11. [Development Workflow](#development-workflow)

## Project Overview

Gemini CLI Desktop is a powerful, cross-platform desktop and web application that provides a modern UI for **Gemini CLI**, **Qwen Code**, and **LLxprt Code**. Built with Rust (Tauri) and React/TypeScript, it enables structured interaction with AI models through the Agent Communication Protocol (ACP).

### Key Features
- **Dual deployment modes**: Native desktop app and web application
- **Real-time communication**: WebSocket-based event system for live updates
- **Tool call confirmation**: User approval workflow for AI agent actions
- **Multi-backend support**: Gemini CLI, Qwen Code, and LLxprt Code integration with support for 9+ AI providers (Anthropic, OpenAI, OpenRouter, Gemini, Qwen, Groq, Together, xAI, and custom endpoints)
- **Project management**: Session-based workspace management with chat history
- **Security-first design**: Comprehensive command filtering and permission system
- **Internationalization**: Full i18n support with language switching for English, Chinese Simplified, and Traditional Chinese
- **Custom title bar**: Enhanced desktop experience with native window controls
- **About dialog**: Integrated help and version information
- **Resizable sidebar**: Interactive sidebar with drag-to-resize functionality and persistent width settings
- **Cross-platform support**: Windows, macOS, and Linux compatibility
- **File viewing support**: PDF, Excel, image, and text file viewers with syntax highlighting
- **Advanced search**: Full-text search across chat history and projects
- **Settings management**: Comprehensive settings dialog with backend configuration

## Architecture

### Rust Workspace Structure

The project is organized as a Rust workspace with three main crates:

#### **`crates/backend`** - Core Business Logic
- **ACP Protocol** (`acp/mod.rs`) - Complete Agent Communication Protocol implementation
  - JSON-RPC 2.0 based messaging
  - Session lifecycle management (initialize, authenticate, new session)
  - Tool call handling with user confirmation flow
  - Content blocks for text, images, audio, and resources
  - Comprehensive test suite with property-based testing
- **Session Management** (`session/mod.rs`) - CLI process orchestration
  - Multi-backend support: Gemini CLI, Qwen Code, and LLxprt Code
  - LLxprt provider configuration for 9+ AI providers (Anthropic, OpenAI, OpenRouter, Gemini, Qwen, Groq, Together, xAI, custom)
  - Working directory context preservation
  - Process lifecycle management
  - Authentication handling (API keys, Vertex AI, OAuth)
  - Robust JSON parsing with non-JSON line filtering
  - Environment variable cleanup with RAII pattern for security
  - API key masking in logs for security compliance
  - SSRF protection with URL validation
- **Event System** (`events/mod.rs`) - Real-time communication backbone
  - Event emission and broadcasting
  - WebSocket integration
  - Tool call confirmation workflow
- **Security** (`security/mod.rs`) - Command execution protection
  - Whitelist of 100+ safe commands
  - Blacklist of dangerous patterns and operations
  - Cross-platform command validation
- **File System** (`filesystem/mod.rs`) - Safe file operations
  - Directory validation and navigation
  - Home directory detection
  - Volume listing (Windows, macOS, Linux)
- **Projects** (`projects/mod.rs`) - Workspace management
  - Project discovery and metadata
  - Chat history and search functionality
  - SHA256-based project identification
- **Search** (`search/mod.rs`) - Full-text search capabilities
  - Chat content indexing
  - Filtering and ranking algorithms
  - Date range and project-based filtering
- **CLI Integration** (`cli/mod.rs`) - Command line interface management
  - Process spawning and lifecycle management
  - Output parsing and streaming
  - Cross-platform command execution
- **RPC Layer** (`rpc/mod.rs`) - Remote procedure call abstraction
  - Type-safe method definitions
  - Serialization/deserialization handling
  - Error propagation and handling

#### **`crates/server`** - Web Server Implementation
- **Rocket-based REST API** - HTTP endpoints for all backend functionality
- **WebSocket handlers** - Real-time event broadcasting to web clients
- **Static file serving** - Embedded frontend distribution
- **Connection management** - WebSocket lifecycle and error handling
- **Binary target**: `gemini-cli-desktop-web`

#### **`crates/tauri-app`** - Desktop Application
- **Native wrapper** around the React frontend
- **Tauri commands** for system integration
- **Event emission** to frontend via Tauri's event system
- **Cross-platform capabilities** with minimal permissions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Piebald-AI/gemini-cli-desktop](https://github.com/Piebald-AI/gemini-cli-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
