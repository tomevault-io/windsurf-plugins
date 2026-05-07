---
trigger: always_on
description: handleStreamingRequest(w, req)
---

# CLAUDE.md - CC-Bridge Development Guide

## Project Overview

**CC-Bridge** is an HTTP service that provides 100% Anthropic API compatibility while using the official Claude Code CLI under the hood. This preserves OAuth access legitimately without authentication hacks.

**Problem:** Anthropic is shutting down third-party OAuth access. Apps using OAuth tokens directly now fail with:
```
LLM request rejected: This credential is only authorized for use with
Claude Code and cannot be used for other API requests.
```

**Solution:** Wrap `claude -p` (Claude CLI print mode) as an HTTP service. Let the official CLI handle authentication.

---

## Quick Start

### Prerequisites

1. **Claude Code CLI installed**:
   ```bash
   npm install -g claude-code
   # Verify
   claude --version
   ```

2. **Claude Code authenticated**:
   ```bash
   claude setup-token
   # Follow OAuth flow or provide API key
   ```

3. **Go 1.21+**:
   ```bash
   go version
   ```

### Development Setup

```bash
# Clone/create project
cd cc-bridge

# Build
make build

# Run server (default port 8321)
./build/ccbridge

# Or with custom port
./build/ccbridge --port 9000

# Test
curl -X POST http://localhost:8321/v1/messages \
  -H "Content-Type: application/json" \
  -d '{"model": "claude-sonnet-4-20250514", "messages": [{"role": "user", "content": "Hello"}]}'
```

---

## Architecture

```
Client (Anthropic SDK)
  ↓ POST /v1/messages
Go HTTP Server (cc-bridge)
  ↓ spawn("claude", ["-p", ...])
Claude CLI
  ↓ OAuth or API key
Anthropic API
```

**Key insight:** We're not fighting OAuth - we're delegating auth to the official tool that's allowed to use it.

---

## Implementation Guide

### Phase 1: Non-Streaming MVP (10-12 hours)

**Goal:** Basic request/response works with Anthropic SDK

#### File Structure
```
cc-bridge/
├── src/
│   ├── main.go          # HTTP server entry point
│   ├── handler.go       # Request handler
│   ├── types.go         # Type definitions
│   ├── cli.go           # Execute claude -p
│   ├── mapper.go        # Map API params ↔ CLI flags
│   ├── streaming.go     # SSE streaming support
│   └── *_test.go        # Tests
├── build/               # Compiled binaries
├── Makefile             # Build commands
└── go.mod
```

#### main.go
```go
package main

import (
    "log"
    "net/http"
    "os"
)

func main() {
    port := os.Getenv("CC_BRIDGE_PORT")
    if port == "" {
        port = "8080"
    }

    http.HandleFunc("/v1/messages", handleMessages)
    http.HandleFunc("/health", handleHealth)

    log.Printf("CC-Bridge starting on port %s", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}

func handleHealth(w http.ResponseWriter, r *http.Request) {
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("OK"))
}
```

#### handler.go
```go
func handleMessages(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }

    // 1. Parse Anthropic request
    var req AnthropicRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        writeErrorResponse(w, "invalid_request_error", err.Error())
        return
    }

    // 2. Check if streaming
    if req.Stream {
        handleStreamingRequest(w, req)
        return
    }

    // 3. Execute CLI
    cliResp, err := executeCLI(req)
    if err != nil {
        writeErrorResponse(w, "api_error", err.Error())
        return
    }

    // 4. Format response
    anthropicResp := formatAnthropicResponse(cliResp, req)

    // 5. Return JSON
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(anthropicResp)
}
```

#### request.go
```go
type AnthropicRequest struct {
    Model       string          `json:"model"`
    Messages    []Message       `json:"messages"`
    System      string          `json:"system,omitempty"`
    MaxTokens   int             `json:"max_tokens,omitempty"`
    Temperature float64         `json:"temperature,omitempty"`
    Tools       []Tool          `json:"tools,omitempty"`
    Stream      bool            `json:"stream,omitempty"`
}

type Message struct {
    Role    string `json:"role"`
    Content string `json:"content"`
}

type Tool struct {
    Name        string                 `json:"name"`
    Description string                 `json:"description"`
    InputSchema map[string]interface{} `json:"input_schema"`
}
```

#### cli.go
```go
import (
    "bytes"
    "encoding/json"
    "os/exec"
    "strings"
)

func executeCLI(req AnthropicRequest) (*CLIResponse, error) {
    // Build command args
    args := []string{"-p"}

    // Model
    args = append(args, "--model", mapModelName(req.Model))

    // Output format
    args = append(args, "--output-format", "json")

    // Temperature
    if req.Temperature > 0 {
        args = append(args, "--temperature", fmt.Sprintf("%.1f", req.Temperature))
    }

    // System prompt
    if req.System != "" {
        args = append(args, "--append-system-prompt", req.System)
    }

    // Tools
    if len(req.Tools) > 0 {
        toolNames := mapToolNames(req.Tools)
        args = append(args, "--tools", strings.Join(toolNames, ","))
    }

    // Build prompt from messages
    prompt := buildPrompt(req.Messages)

    // Execute
    cmd := exec.Command("claude", args...)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ranaroussi/cc-bridge](https://github.com/ranaroussi/cc-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
