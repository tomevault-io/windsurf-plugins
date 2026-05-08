---
trigger: always_on
description: Provides constraints and natural language descriptions for properties:
---

# Apple Foundation Models Framework Documentation

## Overview

The Foundation Models framework is Apple's new API announced at WWDC 2025 that provides developers with direct access to the on-device large language model powering Apple Intelligence. Released to beta on June 9, 2025, this framework enables developers to integrate powerful AI capabilities directly into their apps while maintaining user privacy through on-device processing.

### Key Features

- **On-device processing**: All AI inference runs locally on the device
- **Privacy-focused**: No data leaves the device or is sent to cloud servers
- **Offline capable**: Works without an internet connection
- **Zero cost**: No API fees or cloud computing charges
- **Small footprint**: Built into the OS, doesn't increase app size
- **Swift-native**: Integrates seamlessly with Swift using as few as 3 lines of code

### Platform Availability

- iOS 26
- iPadOS 26
- macOS Tahoe 26
- visionOS 26

### Device Requirements

- iPhone 16 (all models)
- iPhone 15 Pro and iPhone 15 Pro Max
- iPad mini (A17 Pro)
- iPad models with M1 chip or later
- Mac models with M1 chip or later
- Apple Vision Pro

### Language Support

**Available at launch:**
- English, French, German, Italian, Portuguese (Brazil), Spanish, Japanese, Korean, Chinese (simplified)

**Coming by end of 2025:**
- Danish, Dutch, Norwegian, Portuguese (Portugal), Swedish, Turkish, Chinese (traditional), Vietnamese

## The Foundation Model

### Model Specifications

- **Parameters**: ~3 billion
- **Quantization**: 2-bit (with 3.5-3.7 bits-per-weight average using mixed 2-bit and 4-bit configuration)
- **Architecture**: Optimized for Apple Silicon
- **Performance**: 
  - Time-to-first-token latency: ~0.6ms per prompt token
  - Generation rate: 30 tokens per second on iPhone 15 Pro

### Model Capabilities

The on-device model excels at:
- Text summarization
- Entity extraction
- Text understanding and refinement
- Short dialog generation
- Creative content generation
- Classification tasks
- Content tagging
- Natural language search

### Model Limitations

The model is NOT designed for:
- General world knowledge queries
- Advanced reasoning tasks
- Chatbot-style conversations
- Server-scale LLM tasks

## Core Features

### 1. Guided Generation

Guided Generation is the framework's core feature that ensures reliable structured output from the model using Swift's type system.

#### The @Generable Macro

```swift
import FoundationModels

@Generable
struct SearchSuggestions {
    @Guide(description: "A list of suggested search terms", .count(4))
    var searchTerms: [String]
}
```

#### Supported Types

Generable types can include:
- **Primitives**: String, Int, Double, Float, Decimal, Bool
- **Arrays**: [String], [Int], etc.
- **Composed types**: Nested structs
- **Recursive types**: Self-referencing structures

#### The @Guide Macro

Provides constraints and natural language descriptions for properties:

```swift
@Generable
struct Person {
    @Guide(description: "Person's full name")
    var name: String
    
    @Guide(description: "Age in years", .range(0...120))
    var age: Int
    
    @Guide(regex: /^[A-Z]{2}-\d{4}$/)
    var id: String
}
```

#### Basic Usage

```swift
let session = LanguageModelSession()
let prompt = "Generate search suggestions for a travel app"
let response = try await session.respond(
    to: prompt,
    generating: SearchSuggestions.self
)
print(response.content.searchTerms)
```

### 2. Snapshot Streaming

The framework uses a unique snapshot-based streaming approach instead of traditional delta streaming.

#### PartiallyGenerated Types

The @Generable macro automatically generates a `PartiallyGenerated` type with all optional properties:

```swift
@Generable
struct Itinerary {
    var destination: String
    var days: [DayPlan]
    var summary: String
}

// Automatically generates:
// Itinerary.PartiallyGenerated with all optional properties
```

#### Streaming Implementation

```swift
struct ItineraryView: View {
    let session: LanguageModelSession
    @State private var itinerary: Itinerary.PartiallyGenerated?
    
    var body: some View {
        VStack {
            // UI components
            Button("Generate") {
                Task {
                    let stream = session.streamResponse(
                        to: "Plan a 3-day trip to Tokyo",
                        generating: Itinerary.self
                    )
                    
                    for try await partial in stream {
                        self.itinerary = partial
                    }
                }
            }
        }
    }
}
```

#### Best Practices for Streaming

1. **Use SwiftUI animations** to hide latency
2. **Consider view identity** when generating arrays
3. **Property order matters** - properties are generated in declaration order
4. **Place summaries last** for better quality output

### 3. Tool Calling

Tool calling allows the model to execute custom code to retrieve information or perform actions.

#### Defining a Tool

```swift
struct WeatherTool: Tool {
    static let name = "get_weather"
    static let description = "Get current weather for a location"
    
    @Generable
    struct Arguments {
        let city: String
        let unit: String?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ricyoung/OllamaRemote](https://github.com/ricyoung/OllamaRemote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
