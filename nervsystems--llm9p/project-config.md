---
trigger: always_on
description: This guide is for Claude Code and developers working on the llm9p codebase.
---

# llm9p - Development Guide

This guide is for Claude Code and developers working on the llm9p codebase.

## Quick Reference

### Build and Run

```bash
# Build
go build -o llm9p ./cmd/llm9p

# Run with Anthropic API
ANTHROPIC_API_KEY=sk-... ./llm9p -addr :5640

# Run with Claude Max subscription (via CLI)
./llm9p -addr :5640 -backend cli

# Run with debug logging
ANTHROPIC_API_KEY=sk-... ./llm9p -addr :5640 -debug
```

### Testing

```bash
# Run all tests
go test ./...

# Run with coverage
go test -cover ./...

# Test specific package
go test ./internal/protocol/...
go test ./internal/llmfs/...
```

### Development Workflow

```bash
# Install dependencies
go mod tidy

# Format code
go fmt ./...

# Vet code
go vet ./...

# Build
go build -o llm9p ./cmd/llm9p
```

## Architecture Overview

### Project Structure

```
llm9p/
├── cmd/
│   └── llm9p/
│       └── main.go           # Entry point, CLI flags, server setup
├── internal/
│   ├── protocol/             # 9P2000 protocol implementation
│   │   ├── protocol.go       # Message types, constants, encoding
│   │   ├── message.go        # Individual message types
│   │   ├── server.go         # Connection handling
│   │   └── fs.go             # File/Dir interfaces, base implementations
│   ├── llm/                  # LLM client wrapper
│   │   └── client.go         # Anthropic API integration
│   └── llmfs/                # LLM filesystem implementation
│       ├── root.go           # Root directory construction
│       ├── ask.go            # Ask file (shim pattern)
│       ├── state.go          # Model, temperature files
│       ├── system.go         # System prompt file
│       ├── tokens.go         # Read-only token counter
│       ├── new.go            # Conversation reset trigger
│       ├── context.go        # Conversation history
│       ├── example.go        # Usage examples
│       └── stream.go         # Streaming interface
├── go.mod
├── go.sum
├── README.md
└── CLAUDE.md
```

### Key Components

1. **Protocol Layer (`internal/protocol/`)**
   - Implements 9P2000 protocol
   - No external dependencies (stdlib only)
   - `File` and `Dir` interfaces define the filesystem abstraction

2. **LLM Client (`internal/llm/`)**
   - `backend.go` - Backend interface for swappable LLM providers
   - `client.go` - Anthropic API client (requires API key)
   - `cli_client.go` - Claude Code CLI client (uses Max subscription)
   - Manages conversation state
   - Supports both sync and streaming responses
   - Tracks token usage (API only)

3. **LLM Filesystem (`internal/llmfs/`)**
   - Implements each file in the LLM filesystem
   - `AskFile` is the core interaction point
   - State files (`model`, `temperature`) modify client settings
   - `ChunkFile` provides streaming access

## Adding a New File

1. Create a new file in `internal/llmfs/`:

```go
package llmfs

import (
    "github.com/NERVsystems/llm9p/internal/llm"
    "github.com/NERVsystems/llm9p/internal/protocol"
)

type MyFile struct {
    *protocol.BaseFile
    client *llm.Client
}

func NewMyFile(client *llm.Client) *MyFile {
    return &MyFile{
        BaseFile: protocol.NewBaseFile("myfile", 0666),
        client:   client,
    }
}

func (f *MyFile) Read(p []byte, offset int64) (int, error) {
    // Implement read
}

func (f *MyFile) Write(p []byte, offset int64) (int, error) {
    // Implement write
}

func (f *MyFile) Stat() protocol.Stat {
    s := f.BaseFile.Stat()
    // Update s.Length if dynamic
    return s
}
```

2. Add to root directory in `internal/llmfs/root.go`:

```go
root.AddChild(NewMyFile(client))
```

## Protocol Implementation Notes

### Message Flow

1. Client sends T-message (request)
2. Server responds with R-message (response)
3. Each message has a tag for matching requests/responses

### Key 9P Operations

- `Tversion/Rversion` - Protocol negotiation
- `Tattach/Rattach` - Connect to filesystem
- `Twalk/Rwalk` - Navigate directory tree
- `Topen/Ropen` - Open a file
- `Tread/Rread` - Read from file
- `Twrite/Rwrite` - Write to file
- `Tclunk/Rclunk` - Close a fid

### File Interfaces

```go
// File is the interface that files must implement
type File interface {
    Stat() Stat
    Open(mode uint8) error
    Read(p []byte, offset int64) (int, error)
    Write(p []byte, offset int64) (int, error)
    Close() error
}

// Dir extends File with directory operations
type Dir interface {
    File
    Children() []File
    Lookup(name string) (File, error)
}
```

## Debugging

### Enable Debug Logging

```bash
ANTHROPIC_API_KEY=sk-... ./llm9p -debug
```

This logs all 9P messages sent and received.

### Test with 9p Client (plan9port)

```bash
# Using 9pfuse
9pfuse localhost:5640 /mnt/llm

# Using Plan 9's 9p tool (no mount needed)
9p -a localhost:5640 ls llm
9p -a localhost:5640 read llm/model
9p -a localhost:5640 read llm/temperature
9p -a localhost:5640 write llm/ask "What is 2+2?"
9p -a localhost:5640 read llm/ask        # Returns "4"
9p -a localhost:5640 read llm/tokens     # Returns token count

# Multi-turn conversation
9p -a localhost:5640 write llm/ask "Remember the number 42"
9p -a localhost:5640 write llm/ask "What number did I just mention?"
9p -a localhost:5640 read llm/ask        # Returns "42"

# Set system prompt (e.g., persona)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NERVsystems/llm9p](https://github.com/NERVsystems/llm9p) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
