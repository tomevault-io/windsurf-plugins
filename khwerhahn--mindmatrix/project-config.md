---
trigger: always_on
description: MindMatrix is an Obsidian plugin that enhances note-taking by synchronizing documents with a Supabase vector database for AI-powered search. The plugin provides semantic search capabilities through OpenAI embeddings while maintaining data integrity and user experience. It operates entirely within the Obsidian environment, leveraging remote services (Supabase and OpenAI) for data storage and processing.
---

# MindMatrix Architecture

## Overview

MindMatrix is an Obsidian plugin that enhances note-taking by synchronizing documents with a Supabase vector database for AI-powered search. The plugin provides semantic search capabilities through OpenAI embeddings while maintaining data integrity and user experience. It operates entirely within the Obsidian environment, leveraging remote services (Supabase and OpenAI) for data storage and processing.

The primary purpose of MindMatrix is to enable users to create chatbots through n8n that can query and interact with their Obsidian knowledge base. By storing document embeddings in Supabase, the plugin makes the entire vault searchable and accessible to AI-powered applications, allowing users to build custom workflows and chatbots that leverage their personal knowledge base.

The plugin implements a worker actor model with an in-memory event queue to ensure immediate response to user actions while maintaining data consistency. When a user starts typing in Obsidian, the plugin immediately detects these changes and queues them for processing, even while initialization and system startup procedures are running in the background.

## Design Principles

- **Worker Actor Model**: Clear separation of concerns between file event detection and processing
- **Atomic Processing**: Maintaining strict sequential processing to ensure data consistency
- **Offline Resilience**: Robust operation even when connectivity is interrupted
- **Performance**: Minimizing impact on Obsidian while efficiently handling large vaults
- **Reliability**: Comprehensive error handling and recovery mechanisms
- **Testability**: Architecture designed to support comprehensive testing

## Core Components

### 1. Actor System

- **File Watcher Actor**:
  - Hooks into Obsidian file system events
  - Detects file creation, modification, deletion, and moves
  - Generates file change events with metadata
  - Adds events to the in-memory processing queue immediately
  - Operates with minimal latency to capture user changes

- **Startup Scanner Actor**:
  - Activates once during plugin initialization
  - Runs non-blockingly in the background
  - Queries remote database for file statuses and hashes
  - Scans local vault files and calculates hashes
  - Identifies differences between local and remote state
  - Adds files needing synchronization to the queue
  - Respects file exclusion settings

- **Worker Actor**:
  - Runs in a separate Web Worker thread
  - Polls the event queue for new tasks
  - Processes events sequentially to maintain atomicity
  - Calculates file hashes when processing events
  - Handles connectivity to external services
  - Implements retry logic for failed operations
  - Reports progress back to main thread

- **Coordinator**:
  - Manages initialization and shutdown sequences
  - Maintains communication between actors
  - Handles plugin lifecycle events
  - Provides synchronization primitives
  - Manages service connections and state

### 2. Event Queue System

- **Queue Management**:
  - Pure in-memory event queue (no persistence)
  - Strict FIFO (First-In-First-Out) ordering
  - Timestamps for all events to maintain chronology
  - Lightweight implementation for performance
  - Support for priority events (e.g., manual rescans)

- **Event Types**:
  - `FILE_CREATED`: New file detected
  - `FILE_MODIFIED`: Existing file changed
  - `FILE_DELETED`: File removed from vault
  - `FILE_MOVED`: File path changed
  - `RESCAN_REQUESTED`: Manual rescan triggered
  - `VAULT_SCAN`: Full vault scan requested

- **Event Processing States**:
  - `QUEUED`: Added to queue, awaiting processing
  - `PROCESSING`: Currently being processed
  - `COMPLETED`: Successfully processed
  - `FAILED`: Processing failed, pending retry
  - `RETRYING`: Being retried after failure

### 3. File Tracking System

- **File State Management**:
  - Content hash calculation (SHA-256)
  - Path and metadata tracking
  - Change detection based on hash comparison
  - Obsidian metadata extraction (frontmatter, tags, links)
  - Exclusion rule application

- **Hashing Strategy**:
  - Full content hash for change detection
  - Incremental hashing for large files
  - Hash comparison with remote database
  - Hash caching for performance
  - Hash verification on processing

- **Metadata Extraction**:
  - YAML frontmatter parsing
  - Tag detection and normalization
  - Internal link extraction
  - Document structure analysis
  - Content type detection

### 4. Processing Pipeline

- **Initialization**:
  - Plugin startup sequence
  - Service connection establishment
  - Worker thread creation
  - Event listener registration
  - State restoration

- **Document Processing**:
  - File content reading
  - Content chunking with configurable strategies
  - Metadata extraction and normalization
  - OpenAI embedding generation
  - Supabase database synchronization

- **Synchronization**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khwerhahn/MindMatrix](https://github.com/khwerhahn/MindMatrix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
