---
trigger: always_on
description: > **Purpose:** Help AI coding agents understand the Fetchy project structure, conventions, and how to contribute effectively.
---

# AGENTS.md — Fetchy AI Agent Guide

> **Purpose:** Help AI coding agents understand the Fetchy project structure, conventions, and how to contribute effectively.
> **Last Updated:** April 2026

---

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Directory Structure](#directory-structure)
- [Architecture Overview](#architecture-overview)
- [Key Concepts](#key-concepts)
- [Development Workflow](#development-workflow)
- [Code Conventions](#code-conventions)
- [Testing Guidelines](#testing-guidelines)
- [Common Tasks](#common-tasks)
- [Important Files Reference](#important-files-reference)

---

## Project Overview

**Fetchy** is a privacy-focused, self-hosted REST API client built with Electron and React. It's designed as an **AI-native project** — all development is expected to be done using AI coding agents (see [CONTRIBUTING.md](CONTRIBUTING.md)).

### Core Principles

- **Privacy First**: 100% local, no cloud sync, no telemetry
- **AI-Native Development**: All contributions leverage AI agents
- **Offline Capable**: Full functionality without internet
- **Trunk-Based Development**: Short-lived branches merged to `main` quickly

---

## Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| **Frontend** | React | 18.x |
| **Language** | TypeScript | 5.x |
| **Styling** | Tailwind CSS | 3.4.x |
| **State Management** | Zustand | 4.5.x |
| **Code Editor** | CodeMirror | 6.x |
| **Desktop Framework** | Electron | 40.x |
| **Build Tool** | Vite | 4.x |
| **Immutable Updates** | Immer | 10.x |
| **Drag & Drop** | dnd-kit | 6.x |
| **Testing** | Vitest | 0.34.x |
| **Linting** | ESLint | 8.x |
| **Formatting** | Prettier | 3.x |

---

## Directory Structure

```
fetchy/
├── electron/                    # Electron main process (Node.js)
│   ├── main.js                 # Main process entry point
│   ├── preload.js              # Preload scripts for IPC bridge
│   ├── updater.js              # Auto-update logic
│   └── ipc/                    # IPC handlers (modularized)
│       ├── index.js            # IPC handler registration
│       ├── aiHandler.js        # AI provider API calls
│       ├── fileHandlers.js     # File system operations
│       ├── httpHandler.js      # HTTP request proxy
│       ├── jiraHandler.js      # Jira integration
│       ├── secretsHandler.js   # Secure storage (safeStorage)
│       ├── validate.js         # IPC input validation
│       └── workspaceHandler.js # Workspace management
│
├── src/                         # React application (renderer process)
│   ├── App.tsx                 # Main application component
│   ├── main.tsx                # React entry point
│   ├── index.css               # Global styles + Tailwind imports
│   │
│   ├── components/             # React components
│   │   ├── request/            # Request panel sub-components
│   │   │   ├── UrlBar.tsx
│   │   │   ├── BodyEditor.tsx
│   │   │   ├── AuthEditor.tsx
│   │   │   └── ScriptsEditor.tsx
│   │   ├── sidebar/            # Sidebar sub-components
│   │   │   ├── ApiDocsPanel.tsx
│   │   │   ├── HistoryPanel.tsx
│   │   │   ├── SortableCollectionItem.tsx
│   │   │   ├── SortableFolderItem.tsx
│   │   │   ├── SortableRequestItem.tsx
│   │   │   └── SidebarContextMenu.tsx
│   │   ├── openapi/            # OpenAPI editor components
│   │   ├── RequestPanel.tsx    # Main request editor
│   │   ├── ResponsePanel.tsx   # Response viewer
│   │   ├── Sidebar.tsx         # Main sidebar (collections/history/docs)
│   │   ├── AIAssistant.tsx     # AI chat assistant
│   │   ├── EnvironmentModal.tsx
│   │   ├── RunCollectionModal.tsx
│   │   └── ...                 # Other modals and components
│   │
│   ├── store/                  # Zustand state management
│   │   ├── appStore.ts         # Main application state
│   │   ├── workspacesStore.ts  # Multi-workspace management
│   │   ├── preferencesStore.ts # User preferences
│   │   ├── persistence.ts      # Storage adapters (Electron/browser)
│   │   ├── requestTree.ts      # Tree traversal utilities
│   │   ├── entityIndex.ts      # Flat entity index for O(1) lookups
│   │   └── dataMigration.ts    # Data format migrations
│   │
│   ├── hooks/                  # Custom React hooks
│   │   └── useKeyboardShortcuts.ts
│   │
│   ├── types/                  # TypeScript type definitions
│   │   └── index.ts            # Main type exports
│   │
│   └── utils/                  # Utility functions
│       ├── httpClient.ts       # HTTP request execution
│       ├── variables.ts        # Variable substitution (<<var>>)
│       ├── authInheritance.ts  # Auth inheritance resolver
│       ├── codeGenerator.ts    # Code snippet generation
│       ├── curlParser.ts       # cURL command parser
│       ├── postman.ts          # Postman importer
│       ├── hoppscotch.ts       # Hoppscotch importer
│       ├── bruno.ts            # Bruno importer
│       ├── openapi.ts          # OpenAPI importer
│       ├── aiProvider.ts       # AI provider configurations
│       ├── jwt.ts              # JWT decoding utilities
│       └── ...                 # Other utilities
│
├── test/                        # Test files (Vitest)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akineralkan/fetchy](https://github.com/akineralkan/fetchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
