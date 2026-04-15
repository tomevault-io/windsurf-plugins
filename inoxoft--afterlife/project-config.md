---
trigger: always_on
description: This is a Flutter application called "Afterlife" that enables users to interact with AI-powered digital twins of historical figures and create custom AI companions. The app features a hybrid AI architecture supporting both local LLM inference and cloud-based AI services.
---

# Afterlife Flutter Project - Cursor Rules

## Project Overview
This is a Flutter application called "Afterlife" that enables users to interact with AI-powered digital twins of historical figures and create custom AI companions. The app features a hybrid AI architecture supporting both local LLM inference and cloud-based AI services.

## Architecture Patterns

### State Management
- Use Provider pattern for state management
- Create dedicated providers in `lib/features/providers/`
- Always extend `ChangeNotifier` for state classes
- Use `Consumer` and `context.read<T>()` appropriately
- Dispose controllers and subscriptions in provider dispose methods

### Service Architecture
- Services go in `lib/core/services/` for core functionality
- Feature-specific services in `lib/features/[feature_name]/`
- Always implement singleton pattern for services with `static` getInstance methods
- Use dependency injection through Provider for service access
- Initialize services asynchronously with proper error handling

### File Organization
```
lib/
├── core/
│   ├── services/          # Core services (AI, local LLM, hybrid chat)
│   ├── theme/            # App theming and styling
│   ├── utils/            # Utility functions and helpers
│   └── widgets/          # Reusable core widgets
├── features/
│   ├── [feature_name]/
│   │   ├── models/       # Data models for the feature
│   │   ├── providers/    # State management
│   │   ├── widgets/      # Feature-specific widgets
│   │   └── [screens].dart # Screen implementations
├── l10n/                 # Internationalization files
└── main.dart
```

## Coding Conventions

### Naming Conventions
- Files: `snake_case.dart`
- Classes: `PascalCase`
- Variables/functions: `camelCase`
- Constants: `SCREAMING_SNAKE_CASE` or `camelCase` for private
- Folders: `snake_case`
- Character IDs: prefix with `char_` followed by timestamp

### AI Service Integration
- Always use `HybridChatService` for AI interactions
- Support both local and cloud AI models
- Check model type with `CharacterModel.isLocalModel()`
- Implement proper fallback mechanisms
- Use structured prompts with system/user/assistant roles

### Character Model Patterns
```dart
// Always include both system and local prompts
class CharacterModel {
  final String systemPrompt;  // For cloud AI
  final String localPrompt;   // For local AI optimized
  // Use factory constructor for interview data
  factory CharacterModel.fromInterviewData({...})
}
```

### Error Handling
- Wrap AI service calls in try-catch blocks
- Provide user-friendly error messages
- Log debug information with `if (kDebugMode)` guards
- Use `Exception` for recoverable errors
- Implement graceful degradation for AI failures

### Internationalization (i18n)
- All user-facing text must use `AppLocalizations.of(context)`
- Support 9 languages: EN, ES, FR, DE, IT, JA, KO, UK, RU
- Use `UkrainianFontUtils` for Cyrillic language support
- Add language instruction strings to AI prompts for non-English
- Test UI with longer translated strings

## UI/UX Patterns

### Theming
- Use `AppTheme` constants for colors and styles
- Implement cosmic/space theme with dark gradients
- Use `RepaintBoundary` for performance-heavy widgets
- Apply responsive scaling with `ResponsiveUtils.getFontSizeScale()`

### Widget Patterns
```dart
// Standard screen structure
class ExampleScreen extends StatefulWidget {
  @override
  State<ExampleScreen> createState() => _ExampleScreenState();
}

class _ExampleScreenState extends State<ExampleScreen> {
  // Controllers and state variables
  final TextEditingController _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose(); // Always dispose controllers
    super.dispose();
  }
}
```

### Performance Optimizations
- Use `RepaintBoundary` for animated elements
- Implement lazy loading for character lists
- Cache frequently accessed data in providers
- Optimize images with `ImageUtils.pickAndOptimizeImage()`
- Limit particle counts and animation complexity

## AI Integration Guidelines

### Hybrid AI Service
```dart
// Preferred AI interaction pattern
final response = await HybridChatService.sendMessageToCharacter(
  characterId: characterId,
  message: userMessage,
  systemPrompt: character.systemPrompt,
  chatHistory: chatHistory,
  model: character.model,
  localPrompt: character.localPrompt, // For local models
);
```

### Character Card Generation
- Use specific markers: `## CHARACTER CARD SUMMARY ##` and `## END OF CHARACTER CARD ##`
- Extract character names with regex: `## CHARACTER NAME: (.*?) ##`
- Generate 10,000-25,000 token comprehensive character profiles
- Support file processing for PDF, TXT, DOC, DOCX, EML formats

### Local LLM Integration
- Use `flutter_gemma` for local inference
- Support Hammer2.1 model (1.6GB)
- Implement streaming responses for better UX
- Handle model download and initialization gracefully

## Data Persistence

### Character Storage
- Use `SharedPreferences` for character data persistence
- Implement JSON serialization for `CharacterModel`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Inoxoft) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
