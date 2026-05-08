---
trigger: always_on
description: This document provides comprehensive guidelines for AI coding agents (Claude, Cursor, GitHub Copilot, etc.) working on the Front Porch AI codebase. Follow these rules to ensure high-quality, consistent contributions.
---

# AI Agent Guidelines for Front Porch AI

This document provides comprehensive guidelines for AI coding agents (Claude, Cursor, GitHub Copilot, etc.) working on the Front Porch AI codebase. Follow these rules to ensure high-quality, consistent contributions.

## Table of Contents

- [Project Architecture Overview](#project-architecture-overview)
- [Preferred Riverpod Patterns](#preferred-riverpod-patterns)
- [Flutter Widget & State Rules](#flutter-widget--state-rules)
- [Python Sidecar Conventions](#python-sidecar-conventions)
- [Code Style & Naming](#code-style--naming)
- [Files/Areas Requiring Discussion](#filesareas-requiring-discussion)
- [Testing Expectations](#testing-expectations)
- [Examples: Good vs Bad Changes](#examples-good-vs-bad-changes)

## Project Architecture Overview

Front Porch AI is a Flutter desktop application for AI-powered character chat and storytelling. Key architectural components:

### Core Structure
```
lib/
├── main.dart              # App entry point with service initialization
├── app_version.dart       # Version information
├── database/              # Drift database schema and migrations
├── models/                # Data models (characters, chats, etc.)
├── providers/             # State management (currently Provider, migrating to Riverpod)
├── services/              # Business logic and external integrations
├── ui/                    # Flutter UI components
│   ├── dialogs/           # Modal dialogs
│   ├── layout/            # Main app layout
│   ├── pages/             # Screen pages
│   └── widgets/           # Reusable UI components
└── utils/                 # Helper utilities
```

### Key Services
- **ChatService**: Core chat logic and AI integration
- **CharacterRepository**: Character data management
- **EmbeddingSidecar**: Rust-based local embedding server
- **TTSService**: Text-to-speech with multiple engines
- **StorageService**: File and data persistence
- **CloudSyncService**: Google Drive/WebDAV synchronization

### External Dependencies
- **Python sidecars**: TTS (kokoro_tts.py), STT (whisper_stt.py)
- **Rust embedding server**: ONNX-based text embeddings
- **KoboldCpp**: Local LLM backend
- **Database**: SQLite via Drift ORM

## Preferred Riverpod Patterns

**Note**: The codebase currently uses Provider but is migrating to Riverpod. When adding new state management:

### Provider Setup
```dart
@riverpod
class CharacterList extends _$CharacterList {
  @override
  Future<List<Character>> build() async {
    return ref.watch(characterRepositoryProvider).getAllCharacters();
  }

  Future<void> addCharacter(Character character) async {
    await ref.read(characterRepositoryProvider).save(character);
    ref.invalidateSelf();
  }
}
```

### Widget Usage
```dart
class CharacterListWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final characters = ref.watch(characterListProvider);

    return characters.when(
      data: (data) => ListView.builder(...),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

### Best Practices
- Use `AsyncNotifier` for async operations
- Prefer `ref.watch` for reactive dependencies
- Use `ref.read` for one-time actions
- Implement proper error handling with `AsyncValue`

## Flutter Widget & State Rules

### Widget Patterns
- Use `ConsumerWidget` or `StatefulWidget` appropriately
- Prefer composition over inheritance
- Keep widgets focused on single responsibilities
- Use `const` constructors where possible

### State Management
```dart
// Good: Reactive state with Provider
class ChatPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<ChatState>(
      builder: (context, chatState, child) {
        return ListView.builder(
          itemCount: chatState.messages.length,
          itemBuilder: (context, index) => MessageWidget(
            message: chatState.messages[index],
          ),
        );
      },
    );
  }
}
```

### Performance Considerations
- Use `const` for static widgets
- Implement `shouldRebuild` for custom `Consumer`
- Avoid unnecessary rebuilds with `select()`
- Use `AnimatedBuilder` for complex animations

## Python Sidecar Conventions

Python scripts run as subprocesses for TTS, STT, and other media processing.

### Input/Output Protocol
```python
# Read JSON request from stdin
import sys
import json

def main():
    try:
        line = sys.stdin.readline().strip()
        if not line:
            print("No input", file=sys.stderr)
            sys.exit(1)

        request = json.loads(line)

        # Process request...

        # Output result to stdout (JSON)
        result = {"success": True, "data": output_data}
        print(json.dumps(result))

    except Exception as e:
        # Errors to stderr
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

### Error Handling
- Always catch exceptions and exit with non-zero code
- Log errors to stderr, not stdout
- Validate input JSON thoroughly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linux4life1/front-porch-AI](https://github.com/linux4life1/front-porch-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
