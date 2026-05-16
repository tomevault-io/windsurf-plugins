---
trigger: always_on
description: ┌─────────────────────────────────────────────────────────────────┐
---

# Claude Code UI for Kotlin - Technical Architecture & Development Guidelines

## 🏗️ Core Architecture Overview

### High-Level Architecture (Kotlin + Spring Boot + SSE)

```
┌─────────────────────────────────────────────────────────────────┐
│  Frontend: Single Session ID Management (SSE Pattern)           │
│      ↓ SSE Connection (/claude/start, /claude/send)             │
│  Backend: Simple Mapping (SSE_ID → Claude_ID) + Message Model   │
│      ↓ Direct CLI Execution                                     │
│  Claude Code CLI: --resume [ID] or --model sonnet               │
│      ↓ stream-json → ClaudeCodeCliWrapperMessage                │
│  Real-time Response Streaming                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Design Philosophy

This project was inspired by **Claude Code UI** project analysis, implementing the **real-time streaming architecture** using Kotlin + Spring Boot:

1. **Claude Code CLI Process Execution** - `ProcessBuilder` executes Claude Code CLI
2. **JSON Lines Streaming** - Line-by-line JSON message reception from stdout  
3. **Real-time Parsing & Delivery** - Parse each message and deliver via SSE to client
4. **Virtual Thread Utilization** - High-performance processing of blocking I/O

---

## 🚀 Key Implementation Requirements

### Critical System Requirements

- **Backend/Frontend Changes**: When changes occur, forcibly terminate port 8080 and restart before verification
- **Session Continuity**: When sending chat messages from the frontend, backend must use `--resume` command to provide session context to Claude Code CLI
- **Display Modes**: Frontend must distinguish between **[Debug]** mode (raw JSON SSE messages) and **[Normal]** mode (user-friendly processed display)
- **Tool Notification**: In [Normal] mode, all Tool `used` and `result` notifications must be pushed as single SSE messages from Claude Code
- **Tool Name Resolution**: In [Normal] mode, all Tool `result` should not display as "unknown". Especially Bash, TodoWrite, Playwright etc. must display correct tool names
- **UI Styling**: Frontend should match ChatGPT web version's font, font size, line spacing as closely as possible. Avoid excessively small/large text or wide line spacing

---

## 📡 SSE vs WebSocket Architecture Decision

### Why SSE Over WebSocket?

**SSE (Server-Sent Events) Advantages:**
- **Simpler Implementation** - Built-in browser reconnection
- **HTTP/2 Multiplexing** - Better performance over HTTP/2
- **Unidirectional Simplicity** - Perfect for Claude Code CLI streaming use case
- **Automatic Reconnection** - Browser handles connection recovery
- **Easier Debugging** - Standard HTTP requests/responses

**Implementation Pattern:**
```kotlin
// SSE Controller Pattern
@GetMapping("/claude/start", produces = [MediaType.TEXT_EVENT_STREAM_VALUE])
fun startSession(
    @RequestParam prompt: String,
    response: HttpServletResponse
): SseEmitter {
    val emitter = SseEmitter(3600_000L)
    
    // Configure SSE headers
    response.setHeader("Cache-Control", "no-cache")
    response.setHeader("Connection", "keep-alive")
    response.setHeader("X-Accel-Buffering", "no")
    
    // Start Claude Code CLI process
    claudeCodeService.startClaudeCodeCliWrapperSession(sessionId, prompt) { message ->
        emitter.send(
            SseEmitter.event()
                .id("msg-${UUID.randomUUID()}")
                .name("claude-message")
                .data(objectMapper.writeValueAsString(message))
        )
    }
    
    return emitter
}
```

---

## 🔄 Session Management Architecture

### Session ID Mapping Strategy

**Frontend Session Management:**
- Single session ID manages entire conversation flow
- SSE connection reuse handled automatically by backend
- Session ID consistency ensures reconnection works seamlessly

**Backend Session Tracking:**
```kotlin
private val activeEmitters = ConcurrentHashMap<String, SseEmitter>()
private val claudeSessions = ConcurrentHashMap<String, String>()

// Session ID remapping for Claude Code CLI integration
if (claudeSessionId != null && claudeSessionId != tempSessionId) {
    activeEmitters.remove(tempSessionId)
    activeEmitters[claudeSessionId] = emitter
    claudeSessions[claudeSessionId] = claudeSessionId
}
```

### Resume vs New Session Logic

**Command Building Strategy:**
```kotlin
private fun buildClaudeCodeCliWrapperCommand(
    prompt: String,
    claudeSessionId: String? = null,
    isFirstMessage: Boolean = true,
    customOptions: ClaudeCodeCliWrapperOptions? = null
): Array<String> {
    val effectiveResumeSessionId = if (!isFirstMessage && !claudeSessionId.isNullOrBlank()) {
        claudeSessionId
    } else {
        null
    }
    
    val commands = when {
        effectiveResumeSessionId != null -> {
            // Resume existing session
            arrayOf("claude", "--resume", effectiveResumeSessionId, 
                   "--output-format", "stream-json", "--verbose")
        }
        isFirstMessage -> {
            // Start new session
            arrayOf("claude", "--print", prompt, "--output-format", "stream-json", 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JSON-OBJECT/claude-code-ui-kt](https://github.com/JSON-OBJECT/claude-code-ui-kt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
