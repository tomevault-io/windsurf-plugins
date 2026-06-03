---
trigger: always_on
description: This file provides context for AI assistants working on the AI Java Labs project.
---

# AI Java Labs - Development Context

This file provides context for AI assistants working on the AI Java Labs project.

## Project Overview

**AI Java Labs** is an educational Java project demonstrating integration with various AI services and tools. It serves as a teaching repository with hands-on exercises for learning AI API integration patterns in Java.

## Current State

### Recent Updates (May 2026)
- ✅ Updated all AI models to latest versions (local models→gpt-oss, GPT models→gpt-5-nano)
- ✅ Updated LangChain4j to 1.15.0
- ✅ Updated GPT Image examples from deprecated DALL-E 3 to gpt-image-2
- ✅ Configured Java 21 using Gradle's toolchain support
- ✅ Refactored HttpClient usage from try-with-resources to static class attributes
- ✅ Created standalone demo classes for training presentations
- ✅ Implemented test categorization with JUnit tags for cost management
- ✅ Added cleaner streaming patterns using LambdaStreamingResponseHandler
- ✅ Created comprehensive Slidev presentation (40 slides with interactive demos)
- ✅ Added Jackson dependency for JSON parsing with JsonNode
- ✅ Created ResponsesApiDemo (Gson) and ResponsesApiJacksonDemo (Jackson)
- ✅ Fixed Responses API to use 'input' parameter instead of 'messages'
- ✅ Added JSON parsing comparison slides (Gson vs Jackson, JSON Pointer)
- ✅ Fixed slide layout issues including vertical RAG diagram
- ✅ Exported slides to PDF and PPTX formats for training materials
- ✅ Fixed critical bugs (file extension .png→.mp3 in TextToSpeechService)
- ✅ Applied consistent code formatting with Spotless
- ✅ Removed duplicate test methods

### Key Technologies & Versions
- **Java**: 21 (configured via Gradle toolchain, uses records, sealed interfaces, pattern matching)
- **LangChain4j**: 1.15.0 (primary AI framework with streaming enhancements)
- **Gradle**: 8.4+ with Kotlin DSL
- **Slidev**: For interactive presentations
- **JSON Parsing**:
  - Gson 2.14.0 (primary, used throughout)
  - Jackson 2.21.3 (for JsonNode and JSON Pointer support)
- **AI Models**:
  - OpenAI: gpt-4o-mini, gpt-5-nano (Responses API), gpt-image-2, gpt-4o-mini-tts, whisper-1
  - Ollama: gpt-oss (text), moondream (vision)
  - Google: gemini-3-flash-preview

### Project Structure
```
src/main/java/com/kousenit/
├── demos/                     # Standalone demo classes
│   ├── QuickChatDemo.java    # Basic chat example
│   ├── TextToSpeechDemo.java # TTS demonstration
│   ├── MultiModelDemo.java   # Compare multiple providers
│   ├── ToolCallingDemo.java  # LangChain4j tools backed by Java methods
│   ├── StructuredOutputDemo.java # Java records from model output
│   ├── StreamingDemo.java    # Streaming with LambdaStreamingResponseHandler
│   ├── ResponsesApiDemo.java # Responses API with Gson parsing
│   └── ResponsesApiJacksonDemo.java # Responses API with Jackson
├── GptImageService.java       # GPT Image generation
├── EasyRAGDemo.java           # RAG implementation
├── OllamaService.java         # Ollama integration
├── OpenAiService.java         # OpenAI API wrapper
├── TextToSpeechService.java   # TTS implementation
├── Utils.java                 # Utility functions
├── TestCategories.java        # JUnit test tags
├── OllamaRecords.java         # Ollama data models
└── OpenAiRecords.java         # OpenAI data models

Training Materials:
├── slides.md                  # Slidev source (38 interactive slides)
├── slides.pdf                 # PDF export (775 KB)
└── slides.pptx                # PowerPoint export (28 MB)
```

## Development Guidelines

### Code Quality Standards
- **Formatting**: Spotless with Palantir Java Format (auto-applied)
- **Testing**: JUnit 5 with integration tests for external services
- **Error Handling**: Currently uses RuntimeException wrapping (appropriate for teaching)
- **Documentation**: Comprehensive labs.md with step-by-step instructions

### External Dependencies
- **Required**: OpenAI API key (OPENAI_API_KEY environment variable)
- **Optional**: Ollama installation for local AI models
- **Testing**: Some tests require external services to be available

### Build Commands
```bash
./gradlew build          # Full build
./gradlew compileJava    # Compile only (faster for dependency changes)
./gradlew test           # Run tests (some require external services)
./gradlew spotlessApply  # Apply code formatting
```

## Architecture Patterns

### Service Layer
Each AI provider has a dedicated service class:
- **TextToSpeechService**: OpenAI TTS API wrapper
- **OpenAiService**: General OpenAI API operations
- **OllamaService**: Local Ollama model interactions
- **GptImageService**: Image generation with GPT Image

### Data Models
Uses Java records for API data modeling:
- **Sealed interfaces**: `OllamaRequest` with permitted implementations
- **Pattern matching**: Used for request type discrimination
- **JSON mapping**: Gson with field naming policies

### HTTP Client Usage
- Static HttpClient instances for connection pooling and reuse
- Avoided try-with-resources pattern (creates new instances inefficiently)
- Direct exception wrapping in RuntimeException
- Java 21's AutoCloseable HttpClient support acknowledged but not utilized for efficiency

## Common Tasks

### Adding New AI Models
1. Update model constants in service classes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kousen/AiJavaLabs](https://github.com/kousen/AiJavaLabs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
