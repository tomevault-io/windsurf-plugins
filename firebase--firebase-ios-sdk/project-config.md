---
trigger: always_on
description: This file provides a consolidated overview of the `FirebaseAI` directory structure and its contents, intended to help AI agents navigate and understand the codebase efficiently.
---

# FirebaseAI Codebase Map

This file provides a consolidated overview of the `FirebaseAI` directory structure and its contents, intended to help AI agents navigate and understand the codebase efficiently.

---

## Sources/

This directory contains the main source code for the FirebaseAI library.

### Files in `Sources/`

- **`AILog.swift`**: Defines an internal `AILog` enum for logging within the Firebase AI SDK. It includes a `MessageCode` enum for various log messages and helper functions for logging at different levels (error, warning, notice, info, debug).
- **`Chat.swift`**: Defines the `Chat` class, which represents a back-and-forth chat with a `GenerativeModel`. It manages the chat history and provides methods for sending messages.
- **`Constants.swift`**: Defines a `Constants` enum containing constants for the Firebase AI SDK, such as the base error domain.
- **`Errors.swift`**: Defines various error-related structs and enums used for parsing and representing errors from the backend, such as `ErrorStatus` and `RPCStatus`.
- **`FirebaseAI.swift`**: The primary entry point for using the Firebase AI SDK. It provides factory methods for creating `GenerativeModel`, `ImagenModel`, and `LiveGenerativeModel` instances.
- **`FirebaseInfo.swift`**: Encapsulates Firebase-related information used by the SDK, such as project ID, API key, App Check, and Auth interop instances.
- **`GenAIURLSession.swift`**: Provides a `GenAIURLSession` enum with a default `URLSession` instance for the SDK to use.
- **`GenerateContentError.swift`**: Defines the public `GenerateContentError` enum, representing errors that can occur when generating content.
- **`GenerateContentRequest.swift`**: Defines the `GenerateContentRequest` struct, representing a request to generate content from the model.
- **`GenerateContentResponse.swift`**: Represents the model's response to a generate content request, including usage metadata, candidates, and prompt feedback.
- **`GenerationConfig.swift`**: Defines the `GenerationConfig` struct for configuring model parameters (e.g., temperature, topP).
- **`GenerativeAIRequest.swift`**: Defines the `GenerativeAIRequest` protocol for requests sent to the generative AI backend.
- **`GenerativeAIService.swift`**: Responsible for making requests to the generative AI backend, handling authentication, URL construction, and response parsing.
- **`GenerativeModel.swift`**: Defines the `GenerativeModel` class, representing a remote multimodal model. It provides methods for generating content and starting chats.
- **`GenerativeModelSession.swift`**: Provides a simplified interface for single-turn interactions, particularly useful for generating typed objects using the `@Generable` macro.
- **`History.swift`**: A thread-safe class for managing chat history, used by the `Chat` class.
- **`JSONValue.swift`**: Defines the `JSONValue` enum and `JSONObject` typealias for representing JSON values.
- **`ModalityTokenCount.swift`**: Represents token counting information for a single modality.
- **`ModelContent.swift`**: Represents the content of a message to or from the model (can contain multiple `Part`s).
- **`PartsRepresentable.swift`**: Protocol implemented by types that can be converted into an array of `Part`s.
- **`PartsRepresentable+Image.swift`**: Extends `UIImage`, `NSImage`, etc., to conform to `PartsRepresentable`.
- **`Safety.swift`**: Structs and enums related to safety settings and ratings (e.g., `HarmCategory`).
- **`TemplateChatSession.swift`**: Chat session that uses a prompt template.
- **`TemplateGenerateContentRequest.swift`**: Request for generating content from a template.
- **`TemplateGenerativeModel.swift`**: Model for generating content from a prompt template.
- **`TemplateImagenGenerationRequest.swift`**: Request for generating images from a template.
- **`TemplateImagenModel.swift`**: Model for generating images from a prompt template.
- **`TemplateInput.swift`**: Defines the `TemplateInput` enum for representing different types of input to a template.
- **`Tool.swift`**: Structs and enums related to tools and function calling (e.g., `FunctionDeclaration`).

---

## Sources/Protocols/

This directory contains Swift protocols used throughout the FirebaseAI library.

### Public Protocols

- **`ConvertibleFromGeneratedContent.swift`**: Defines `ConvertibleFromGeneratedContent` protocol for types that can be initialized from `GeneratedContent`.
- **`ToolRepresentable.swift`**: Protocol for types that can be represented as a tool in `FirebaseAILogic`.

### Internal Protocols

- **`CodableProtoEnum.swift`**: Provides helper protocols for encoding and decoding protobuf enums (`ProtoEnum`, `DecodableProtoEnum`, `EncodableProtoEnum`).
- **`ConvertibleToGeneratedContent.swift`**: Defines `ConvertibleToGeneratedContent` protocol for internal use.

---

## Sources/Types/

Data types used in the FirebaseAI library.

### Public Types

- **`Backend.swift`**: Used to configure the backend API (Gemini Enterprise Agent Platform or Google AI).
- **`ImageConfig.swift`**: Defines the `ImageConfig` struct, used for configuring generated image properties like aspect ratio and size.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [firebase/firebase-ios-sdk](https://github.com/firebase/firebase-ios-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
