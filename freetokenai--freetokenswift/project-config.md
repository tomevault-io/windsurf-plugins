---
trigger: always_on
description: This guide helps AI assistants (like Claude Code) integrate FreeToken into Swift projects. FreeToken provides hybrid AI capabilities with both on-device and cloud inference, document search (RAG), message threading, and function calling.
---

# FreeToken Swift SDK - AI Assistant Guide

This guide helps AI assistants (like Claude Code) integrate FreeToken into Swift projects. FreeToken provides hybrid AI capabilities with both on-device and cloud inference, document search (RAG), message threading, and function calling.

## Requirements

- **iOS 17+** or **macOS 15+** (also supports watchOS 11+, tvOS 18+, visionOS 2+)
- **Swift Package Manager**
- **FreeToken API Key** from [console.freetoken.ai](https://console.freetoken.ai)

## Installation

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/FreeTokenAI/FreeTokenSwift.git", from: "1.0.0")
]
```

## Core Concepts

- **Hybrid AI**: Automatically routes requests between on-device and cloud AI based on device capabilities
- **Session-Based API**: Modern session management for chat and completion workflows with automatic memory management
- **Message Threads**: Persistent conversation threads stored in the cloud with local caching
- **RAG (Retrieval Augmented Generation)**: Document search integrated into AI responses
- **Tool Calling**: AI can call functions in your app to retrieve context or perform actions
- **Encryption**: Optional client-side encryption for messages and documents

## Quick Start

### 1. Configuration

```swift
import FreeToken

// Basic configuration
try FreeToken.shared.configure(appToken: "your-api-key")

// With encryption (optional)
try FreeToken.shared.configure(
    appToken: "your-api-key",
    sharedPublicEncryptionKey: "public-key",  // For public documents
    userPrivateEncryptionKey: "private-key",  // For messages & private docs
    logLevel: .info
)
```

### 2. Device Registration

Required before using AI features. Determines device capabilities and initializes models.

```swift
await FreeToken.shared.registerDeviceSession(
    scope: "my-app-v1",
    success: {
        print("Device registered successfully")
    },
    error: { error in
        print("Registration failed: \(error)")
    }
)
```

### 3. Download AI Model

```swift
await FreeToken.shared.downloadAIModel(
    modelCode: nil,  // Use default model
    success: { state in
        switch state {
        case .downloaded:
            print("Model ready for local inference")
        case .aiNotSupported:
            print("Device not capable - will use cloud")
        }
    },
    error: { error in
        print("Download failed: \(error)")
    },
    progressPercent: { progress in
        print("Progress: \(Int(progress * 100))%")
    }
)
```

---

## API Reference

### Session-Based Chat API

The session-based API provides better resource management, preloading, and stateful conversation handling.

#### Get Chat Session

Creates a new chat session with automatic local/cloud routing:

```swift
// Automatic routing (local preferred, cloud fallback)
let session = try await FreeToken.shared.getChatSession()

// Force cloud-only
let cloudSession = try await FreeToken.shared.getChatSession(
    runLocation: .cloudRun
)

// Force local-only
let localSession = try await FreeToken.shared.getChatSession(
    runLocation: .localRun
)

// With existing message thread
let session = try await FreeToken.shared.getChatSession(
    messageThreadID: "thread-id"
)

// With specific model
let session = try await FreeToken.shared.getChatSession(
    modelCode: "llama-3.2-1b"
)

// With tool access control
let session = try await FreeToken.shared.getChatSession(
    toolAccess: [.allowSpecific(["get_weather", "search_docs"])]
)
```

**Important:** `getChatSession()` automatically determines whether to use local or cloud execution based on:
- Model download status
- Device capabilities
- Available memory
- `runLocation` parameter

#### Create Message Thread

```swift
// Create a new thread for the session
let thread = try await session.createMessageThread()
print("Thread ID: \(thread.id)")  // Save this ID for future use
```

**Note:** Each chat session can only have one thread. Create new sessions for new conversations.

#### Add Message

```swift
// Add user message
let userMsg = FreeToken.Message(role: .user, content: "What is quantum computing?")
let addedMsg = try await session.addMessage(message: userMsg)
```

#### Generate AI Response

```swift
// Simple generation
let response = try await session.generateNewMessage()
print("AI: \(response.content)")

// With streaming
let response = try await session.generateNewMessage(
    chatStatusStream: { token, status in
        if let token = token {
            print(token, terminator: "")
        }

        switch status {
        case .starting:
            print("[Starting]")
        case .streaming_tokens:
            break  // Printing tokens above
        case .stream_ended:
            print("\n[Complete]")
        case .cloud_fallback:
            print("[Falling back to cloud]")
        default:
            print("[\(status.rawValue)]")
        }
    }
)

// With document search (RAG)
let response = try await session.generateNewMessage(
    documentSearchScope: "product-docs",
    privateDocumentStoreIDs: ["store-id-1"]
)

// With tool/function calling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FreeTokenAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
