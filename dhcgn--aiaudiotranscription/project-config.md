---
trigger: always_on
description: Android app for transcribing media files using OpenAI's Whisper API and GPT-4o models with AI-powered text cleanup. Built with Kotlin, Jetpack Compose, and Hilt for dependency injection.
---

# Copilot Instructions: AI Transcription App

Android app for transcribing media files using OpenAI's Whisper API and GPT-4o models with AI-powered text cleanup. Built with Kotlin, Jetpack Compose, and Hilt for dependency injection.

## Build & Test Commands

### Building
```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Clean build (required after changing local.properties)
./gradlew clean assembleDebug
```

### Testing
```bash
# Run all unit tests
./gradlew test

# Run specific test class
./gradlew test --tests app.hdev.io.aitranscribe.ClipboardHelperTest

# Run instrumented tests (requires emulator/device)
./gradlew connectedAndroidTest
```

### Development with Pre-configured API Key
For testing builds with embedded OpenAI API keys, see [TESTING.md](../TESTING.md). Add `OPENAI_API_KEY=sk-...` to `local.properties` and run `./gradlew clean assembleDebug`.

## Architecture Overview

### Core Components

**Presentation Layer** (`presentation/`)
- `MainActivity`: Main transcription UI with file selection, processing states, and result display
- `SettingsActivity`: API key management, model selection, language/prompt configuration
- `HistoryActivity`: Local transcription history with statistics and search
- Built with Jetpack Compose; state managed directly in Activities (ViewModels not yet implemented)

**Data Layer** (`data/`)
- `TranscriptionDbHelper`: SQLiteOpenHelper for local history storage
- `TranscriptionEntry`: Data model with transcript text, settings, and statistics (file sizes, duration, character count)
- **Migration Note**: Room migration is planned but not yet implemented

**API Layer** (`api/`)
- `OpenAiApiService`: Retrofit service interface for OpenAI API
- `RetrofitClient`: Creates Retrofit instance with OkHttp logging interceptor
- Supports three models: `whisper-1`, `gpt-4o-audio-preview-transcribe`, `gpt-4o-mini-audio-preview-transcribe`

**Utils** (`utils/`)
- `FileProcessingManager`: Handles media file copying, conversion to M4A/AAC using Media3 Transformer
- `ClipboardHelper`: Formats transcription history entries for clipboard with statistics
- Uses Hilt `@Singleton` and `@Inject` for dependency injection

**DI Module** (`di/AppModule`)
```kotlin
@Provides @Singleton
fun provideWhisperApiService(@ApplicationContext context: Context): OpenAiApiService
fun provideTranscriptionDbHelper(@ApplicationContext context: Context): TranscriptionDbHelper
```

**Security** (`sharedPrefsUtils/SharedPrefsUtils`)
- API keys stored in `EncryptedSharedPreferences` with AES256_GCM encryption
- BuildConfig mechanism for embedding test API keys from `local.properties`

### Key Workflows

**File Processing Pipeline**
1. User selects/shares media file → URI received
2. `FileProcessingManager.processAudioFile(uri)` copies and converts to M4A/AAC format
3. File size validated (≤24MB) before upload
4. Returns `ProcessingResult` with file metadata (original/processed sizes, filename)

**Transcription Flow**
1. Processed file uploaded via Retrofit multipart request
2. Model-specific API endpoint called (Whisper or GPT-4o variants)
3. Optional AI cleanup using GPT-4o chat completions with custom prompt
4. Result stored in SQLite with comprehensive statistics
5. UI displays transcript with copy/share/retry options

**Auto-Format Feature**
- When enabled in Settings, automatically runs AI cleanup after transcription
- Uses configurable cleanup prompt to enhance readability while preserving content

## Project Conventions

### Dependency Injection
- **Always use Hilt**: Inject `OpenAiApiService`, `TranscriptionDbHelper`, `FileProcessingManager`
- Don't create instances manually (e.g., `RetrofitClient.create()` in Activities)
- Use `@AndroidEntryPoint` on Activities that need injection

### Concurrency Patterns
- Network calls: Use Retrofit's `suspend` functions with `lifecycleScope.launch`
- File operations: Wrap in `withContext(Dispatchers.IO)`
- UI updates: Ensure main thread dispatch after background operations
- Use `rememberCoroutineScope()` in Composables for async operations

### File Processing
- Audio conversion: Media3 Transformer with AAC encoder (adaptive bitrate: 16-32kbps)
- Output format: Always M4A container with AAC codec
- Temp files: Use `context.filesDir` for intermediate files, clean up on error
- File naming: Avoid hardcoded names; use unique identifiers to prevent conflicts

### Database Operations
- All queries on background thread (SQLiteOpenHelper is sync-blocking)
- Schema migrations: Use `onUpgrade()` with version checks
- Statistics tracked: `originalFileSizeBytes`, `uploadedFileSizeBytes`, `transcriptLength`, `audioDurationSeconds`

### API Key Management
- Runtime storage: `SharedPrefsUtils.getApiKey(context)` / `saveApiKey(context, key)`
- Build-time embedding: `BuildConfig.DEFAULT_OPENAI_API_KEY` from `local.properties`
- Initialization: `AITranscriptionApp.onCreate()` transfers BuildConfig key to encrypted storage if not set
- Never log or expose API keys in error messages

### Testing Infrastructure
- Unit tests: `app/src/test/` with JUnit 4

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dhcgn/AIAudioTranscription](https://github.com/dhcgn/AIAudioTranscription) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
