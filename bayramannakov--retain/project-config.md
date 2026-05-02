---
trigger: always_on
description: Native macOS app that aggregates AI conversations from multiple platforms into a unified, searchable knowledge base with intelligent learning extraction.
---

# Retain

Native macOS app that aggregates AI conversations from multiple platforms into a unified, searchable knowledge base with intelligent learning extraction.

**GitHub:** https://github.com/BayramAnnakov/retain

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         SwiftUI                              │
│  ContentView → ConversationDetailView, MenuBarView, etc.    │
├─────────────────────────────────────────────────────────────┤
│                    AppState (@MainActor)                     │
│  - conversations, searchResults, syncState                   │
│  - manages all UI state and coordinates services             │
├─────────────────────────────────────────────────────────────┤
│                        Services                              │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────────┐   │
│  │ SyncService │  │ FileWatcher │  │  WebSyncEngine     │   │
│  │ (actor)     │  │ (FSEvents)  │  │  (claude.ai/gpt)   │   │
│  └──────┬──────┘  └──────┬──────┘  └─────────┬──────────┘   │
│         └────────────────┴───────────────────┘              │
│                          │                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  ConversationRepository / LearningRepository (GRDB)     ││
│  └─────────────────────────────────────────────────────────┘│
│                          │                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │  SQLite + FTS5 (~/.../Retain/retain.sqlite)            ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

```
Sources/Retain/
├── App/                    # Entry point, AppState, ContentView
├── Components/             # Reusable UI (SyncOverlay, ProviderBadge)
├── Data/
│   ├── Models/             # Conversation, Message, Learning, Provider
│   ├── Parsers/            # ClaudeCodeParser, CodexParser
│   └── Storage/            # Database, Repositories
├── Features/
│   ├── ConversationBrowser/
│   ├── Learning/
│   ├── MenuBar/
│   ├── Settings/
│   └── Onboarding/
├── Integrations/           # macOS native integrations
│   ├── SpotlightIndexer.swift  # Core Spotlight indexing
│   └── URLSchemeHandler.swift  # retain:// deep linking
├── Intents/                # App Intents for Shortcuts/Siri
│   ├── ConversationEntity.swift
│   ├── ConversationQuery.swift
│   ├── RetainIntents.swift
│   └── ShortcutsProvider.swift
└── Services/
    ├── FileWatcher.swift   # FSEvents for file changes
    ├── SyncService.swift   # Background sync (actor)
    ├── Learning/           # CorrectionDetector, CLAUDEMDExporter
    ├── Search/             # SemanticSearch, OllamaService
    └── WebSync/            # ClaudeWebSync, ChatGPTWebSync
```

## Key Components

### Data Sources

| Source | Location | Format | Auto-sync |
|--------|----------|--------|-----------|
| Claude Code | `~/.claude/projects/**/*.jsonl` | JSONL | Yes (FSEvents) |
| Codex CLI | `~/.codex/history.jsonl` | JSONL | Yes (FSEvents) |
| Cursor | `~/Library/Application Support/Cursor/User/*Storage/state.vscdb` | SQLite | Yes (FSEvents) |
| claude.ai | Web API | JSON | Manual connect |
| chatgpt.com | Web API | JSON | Manual connect |

### Threading Model

- **SyncService**: Swift `actor` for thread-safe background sync
- **AppState**: `@MainActor` for UI state management
- **Database ops**: `DispatchQueue.global(qos: .userInitiated)`
- **UI updates**: Batched progress updates to reduce MainActor thrashing

### Core Models

```swift
struct Conversation {
    var id: UUID
    var provider: Provider       // .claudeCode, .claudeWeb, .chatgptWeb, .codex
    var sourceType: SourceType   // .cli, .web, .importFile
    var externalId: String?      // For deduplication
    var title: String?
    var projectPath: String?     // For CLI sources
    var messageCount: Int
}

struct Message {
    var id: UUID
    var conversationId: UUID
    var role: Role              // .user, .assistant, .system, .tool
    var content: String
    var timestamp: Date
}

struct Learning {
    var id: UUID
    var conversationId: UUID
    var type: LearningType      // .correction, .positive, .implicit
    var extractedRule: String
    var status: LearningStatus  // .pending, .approved, .rejected
    var scope: LearningScope    // .global, .project
}
```

## macOS Native Integration

### Spotlight Integration
Conversations are indexed in system Spotlight for universal search.
- **SpotlightIndexer** (`Integrations/SpotlightIndexer.swift`): Actor that manages Core Spotlight indexing
- Automatically indexes conversations after sync
- Toggle in Settings → Diagnostics → Spotlight Integration
- Test with: `mdfind "kMDItemContentType == 'com.retain.conversations'"`

### URL Scheme (Deep Linking)
Retain supports `retain://` URLs for deep linking:

| URL | Action |
|-----|--------|
| `retain://conversation/{uuid}` | Open specific conversation |
| `retain://search?q={query}` | Search with query |
| `retain://learnings` | Open learnings view |
| `retain://sync` | Trigger sync |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BayramAnnakov/retain](https://github.com/BayramAnnakov/retain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
