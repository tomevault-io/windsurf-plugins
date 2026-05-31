---
trigger: always_on
description: > **Note for Claude:** This file is the **Single Source of Truth** for the `louter` project architecture, testing strategy, and coding standards. Read this before modifying any code.
---

# CLAUDE.md

> **Note for Claude:** This file is the **Single Source of Truth** for the `louter` project architecture, testing strategy, and coding standards. Read this before modifying any code.

## 1. Project Overview

**Louter** is a Haskell library and proxy server for multi-protocol LLM communication.

### Dual Purpose

1. **Client Library** - Import into your Haskell application to connect to any LLM API
2. **Proxy Server** - Run standalone to bridge between different LLM API protocols

### Supported APIs

**Frontends (What clients can send):**
- OpenAI API format
- Anthropic (Claude) API format
- Google Gemini API format

**Backends (What louter can connect to):**
- OpenAI API, Anthropic API, Google Gemini API
- Self-hosted/OSS models: llama-server, vLLM, text-generation-inference
- Local models via llama.cpp server

### Key Features

- **Protocol Translation**: Automatic conversion between API formats
- **Streaming Support**: Server-Sent Events (SSE) with proper buffering
- **Function Calling**: Tool/function calling across all protocols
- **Vision Support**: Multimodal image handling
- **XML Tools**: Support for Qwen's XML-based function calling format
- **Configurable Auth**: Optional authentication for local vs cloud backends

---

## 2. Haskell Implementation Architecture

### Overview

**Key Design Principle:** *"Louter is a protocol converter. The API can connect to OpenAI API and Gemini API like the proxy."*

**Single Unified Implementation: `louter`**

The Haskell `louter` package serves dual purposes:

1. **As a Library** - Import into your Haskell application to connect to any LLM API
   ```haskell
   import Louter.Client

   -- Connect to Gemini API using OpenAI-style requests
   client <- newClient GeminiBackend "gemini-api-key"
   response <- chatCompletion client openAIStyleRequest
   ```

2. **As a Proxy Server** - Run standalone to proxy requests between protocols
   ```bash
   stack run louter-server -- --config config.yaml --port 9000
   ```

**Core Components:**
- Protocol converters (OpenAI ↔ Gemini ↔ Anthropic)
- SSE parser with attoparsec
- Delta tokenizer/classifier
- Stateful function call buffering
- Mock server for testing (`openai-mock`)

### 2.1 SSE Parsing with Attoparsec

The library uses `attoparsec` for efficient incremental SSE parsing.

**SSE Format:**
```
data: {"id":"chatcmpl-...","choices":[{"delta":{...}}]}\n
\n
```

**Parser Structure:**
```haskell
data SSEChunk = SSEChunk
  { sseData :: ByteString  -- Raw JSON payload
  , sseEvent :: Maybe Text -- Optional event type
  } | SSEDone              -- [DONE] marker

parseSSE :: Parser SSEChunk
parseSSE = choice
  [ string "data: [DONE]" >> pure SSEDone
  , SSEChunk <$> (string "data: " *> takeTill isEndOfLine) <*> pure Nothing
  ]
```

### 2.2 Delta Type Classification

The tokenizer identifies delta types by examining the JSON structure:

```haskell
data DeltaType
  = ReasoningDelta Text       -- delta.reasoning: "thinking tokens"
  | ContentDelta Text         -- delta.content: "response text"
  | ToolCallDelta ToolCallFragment  -- delta.tool_calls[]: function call
  | RoleDelta Role            -- delta.role: "assistant"
  | FinishDelta FinishReason  -- finish_reason: "stop" | "tool_calls"
  | EmptyDelta                -- delta: {}

data ToolCallFragment = ToolCallFragment
  { tcfIndex :: Int
  , tcfId :: Maybe Text
  , tcfName :: Maybe Text
  , tcfArguments :: Maybe Text
  }

classifyDelta :: Value -> Either String DeltaType
classifyDelta = withObject "delta" $ \obj -> do
  case HM.lookup "delta" obj of
    Just (Object delta) ->
      case HM.lookup "reasoning" delta of
        Just (String txt) -> pure $ ReasoningDelta txt
        Nothing -> case HM.lookup "content" delta of
          Just (String txt) -> pure $ ContentDelta txt
          Nothing -> case HM.lookup "tool_calls" delta of
            Just (Array calls) -> ... -- Parse tool call fragment
            Nothing -> pure EmptyDelta
```

### 2.3 Buffering Strategy

**Key Principle:** Different delta types require different streaming behaviors.

| Delta Type | Buffering | Reason |
|------------|-----------|--------|
| `reasoning` | **No buffer** - stream immediately | User sees thinking process in real-time |
| `content` | **No buffer** - stream immediately | User sees response text as it generates |
| `tool_calls` | **Buffer until complete** | Must assemble valid JSON before emitting |
| `role` | **Pass through** | Metadata, no buffering needed |
| `finish_reason` | **Pass through** | End-of-stream marker |

**State Machine:**
```haskell
data StreamState = StreamState
  { ssToolCalls :: Map Int ToolCallState  -- Track by index
  , ssMessageId :: Text
  }

data ToolCallState = ToolCallState
  { tcsId :: Text
  , tcsName :: Text
  , tcsArguments :: Builder  -- Efficient text accumulation
  , tcsComplete :: Bool
  }

processChunk :: StreamState -> DeltaType -> (StreamState, [OutputChunk])
processChunk state (ContentDelta txt) =
  -- Immediate emission, no buffering
  (state, [OutputContentChunk txt])

processChunk state (ToolCallDelta frag) =
  -- Buffer arguments, emit only when JSON is complete

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junjihashimoto/louter](https://github.com/junjihashimoto/louter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
