---
trigger: always_on
description: **Last Updated**: 2025-10-26 17:30:00 UTC
---

# AGENTS.md - ZigNet Project Specification

**Last Updated**: 2025-10-26 17:30:00 UTC  
**Status**: Phase 4 COMPLETE - Ready for Deployment  
**Owner**: fulgidus  
**Repository**: https://github.com/fulgidus/zignet

---

## 1. PROJECT OVERVIEW

### Mission
ZigNet is an **MCP (Model Context Protocol) Server** that integrates with Claude (or other LLMs) to provide intelligent Zig code analysis, validation, and assistance.

### Purpose
- Enable Claude to analyze Zig code without leaving the chat
- Validate Zig syntax and type correctness in real-time
- Generate corrected Zig code based on validation results
- Provide Zig documentation context for advanced features

### Use Case
**Developer using Claude:**
```
User: "Analyze this Zig code for me"
      [pastes Zig code]
↓
Claude: "I'll use ZigNet to analyze this"
↓
Claude calls: ZigNet MCP /analyze_zig
↓
ZigNet: "Type error at line 5: i32 cannot be string"
↓
Claude: "Here's the fix: change x to u32"
```

### Target Users
- Zig developers using Claude
- Advanced Zig projects (NOT beginner tutorials)
- Production-grade code validation

---

## 2. ARCHITECTURE

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│ Claude (Claude.ai / Claude API)                             │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ MCP Protocol (JSON-RPC)
                     │
┌────────────────────▼────────────────────────────────────────┐
│ ZigNet MCP Server (Node.js/TypeScript)                      │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ MCP Handler Layer                                    │   │
│  │ - Tool routing                                       │   │
│  │ - Request validation                                │   │
│  │ - Response formatting                               │   │
│  └──────────────────────────────────────────────────────┘   │
│                     ↓                                        │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Analysis Engine (Core Logic)                         │   │
│  │                                                      │   │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │   │
│  │  │  Lexer     │→ │  Parser    │→ │ TypeChecker  │  │   │
│  │  │ (Tokenize) │  │ (Build AST)│  │ (Validate)   │  │   │
│  │  └────────────┘  └────────────┘  └──────────────┘  │   │
│  │                                                      │   │
│  │  ┌────────────┐  ┌────────────┐  ┌──────────────┐  │   │
│  │  │ CodeGen    │  │ ErrorMap   │  │ DocLookup    │  │   │
│  │  │ (Emit Code)│  │ (Error DB) │  │ (Zig Docs)   │  │   │
│  │  └────────────┘  └────────────┘  └──────────────┘  │   │
│  │                                                      │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Component Interaction

```
INPUT: Zig source code
  ↓
[LEXER] Tokenize → Token[]
  ↓
[PARSER] Parse tokens → AST
  ↓
[TYPE_CHECKER] Validate types → Typed AST or Error[]
  ↓
IF errors:
  [ERROR_MAP] Format error → Error Report
  ELSE:
  [CODEGEN] Generate clean Zig → Zig String
  ↓
[FORMATTER] Pretty-print → Final Output
  ↓
RESPONSE: JSON to Claude
```

---

## 3. MCP SERVER DESIGN

### Server Entry Point

```typescript
// src/mcp-server.ts
const server = new Server({
  name: "zignet",
  version: "0.1.0",
});

// Register tools available to Claude
server.setRequestHandler(CallToolRequestSchema, handleToolCall);
```

### Exposed Tools

#### Tool 1: `analyze_zig`
**Purpose**: Analyze Zig code for errors and validation

**Request**:
```json
{
  "method": "tools/call",
  "params": {
    "name": "analyze_zig",
    "arguments": {
      "code": "fn add(a: i32, b: i32) i32 {\n  return a + b;\n}",
      "action": "lint"
    }
  }
}
```

**Response (Success)**:
```json
{
  "content": [
    {
      "type": "text",
      "text": "✅ Analysis Result:\n- Syntax: Valid\n- Type Check: PASS\n- Warnings: 0\n- Errors: 0"
    }
  ]
}
```

**Response (Error)**:
```json
{
  "content": [
    {
      "type": "text",
      "text": "❌ Type Error:\nCannot assign string to i32\nExpected: i32\nGot: []const u8\n\nSuggestion: Change variable type to []const u8 or convert string to i32"
    }
  ]
}
```

---

#### Tool 2: `compile_zig`
**Purpose**: Compile Zig code and generate output

**Request**:
```json
{
  "method": "tools/call",
  "params": {
    "name": "compile_zig",
    "arguments": {
      "code": "fn add(a: i32, b: i32) i32 { return a + b; }",
      "output_format": "zig"
    }
  }
}
```

**Response**:
```json
{
  "content": [
    {
      "type": "text",
      "text": "✅ Compiled successfully:\n\n```zig\nfn add(a: i32, b: i32) i32 {\n    return a + b;\n}\n```"
    }
  ]
}
```

---

#### Tool 3: `get_zig_docs`
**Purpose**: Retrieve Zig documentation for specific topics

**Request**:
```json
{
  "method": "tools/call",
  "params": {
    "name": "get_zig_docs",
    "arguments": {
      "topic": "comptime",
      "detail_level": "advanced"
    }
  }
}
```

**Response**:
```json
{
  "content": [
    {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fulgidus/zignet](https://github.com/fulgidus/zignet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
