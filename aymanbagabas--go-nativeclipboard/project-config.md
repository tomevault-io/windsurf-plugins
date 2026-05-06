---
trigger: always_on
description: This file documents everything an AI agent needs to know to work effectively in this codebase.
---

# AGENTS.md

This file documents everything an AI agent needs to know to work effectively in this codebase.

## Project Overview

**go-nativeclipboard** is a Go library that provides native clipboard functionality using [purego](https://github.com/ebitengine/purego) instead of cgo. This enables clipboard access without requiring a C compiler.

- **Language**: Go 1.25.4
- **Module**: `github.com/aymanbagabas/go-nativeclipboard`
- **Key Dependencies**: 
  - `github.com/ebitengine/purego` - For calling native APIs without cgo
- **Status**: macOS, Linux, FreeBSD, and Windows implementations complete

## Project Structure

```
.
├── clipboard.go          # Main API and public interfaces
├── clipboard_darwin.go   # macOS implementation (NSPasteboard via purego)
├── clipboard_x11.go      # X11 implementation for Linux and FreeBSD (via purego)
├── clipboard_windows.go  # Windows implementation (Win32 API via syscall)
├── clipboard_test.go     # Platform-agnostic tests
├── doc.go               # Package documentation for pkg.go.dev
├── go.mod               # Go module definition
├── go.sum               # Dependency checksums
├── README.md            # User-facing documentation
├── AGENTS.md            # This file
└── examples/            # Example programs
```

## Dependencies

The project uses purego to call native operating system APIs without cgo:

1. **github.com/ebitengine/purego** - Pure Go library for calling C functions and native APIs
   - Used to call Objective-C runtime on macOS
   - Will be used for X11/Wayland on Linux and Win32 on Windows

Previous dependencies (from golang.design/x/clipboard) have been removed as they used cgo.

## Essential Commands

### Building
```bash
go build ./...
```

### Testing
```bash
# Run all tests on Linux/BSD (requires X11 and Xvfb for headless)
go test ./...

# Run specific test
go test -v -run TestWriteReadText

# Run with coverage
go test -cover ./...

# Run with race detector
go test -race ./...

# For headless Linux/BSD testing
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
export DISPLAY=:99.0
go test -v ./...
```

### Module Management
```bash
# Download dependencies
go mod download

# Update purego to latest
go get -u github.com/ebitengine/purego
go mod tidy
```

## Code Organization

### Package Structure

- **Package name**: `nativeclipboard`
- **Single package**: No subpackages
- **Platform-specific code**: Build tags separate implementations

### API Design

The public API is in `clipboard.go`:
- `Format` type with `Text` and `Image` constants
- `Format.Read()` - Read clipboard data
- `Format.Write([]byte)` - Write clipboard data  
- `Format.Watch(context.Context)` - Monitor clipboard changes

Usage: `nativeclipboard.Text.Read()`, `nativeclipboard.Image.Write(data)`, etc.

All methods return errors. The package initializes automatically via `func init()`.

### Build Tags

Platform-specific files use build constraints:
```go
//go:build darwin && !ios
//go:build (linux || freebsd) && !android
//go:build windows
```

### X11 Implementation (Linux and FreeBSD - Complete)

Uses purego to call X11 library functions directly:

**Note**: Only FreeBSD is supported among BSD systems, as purego officially supports FreeBSD only.

1. **Dynamic library loading**:
   - Dynamically loads libX11.so using `purego.Dlopen`
   - Searches common paths: libX11.so.6, libX11.so
   - FreeBSD-specific paths: /usr/local/lib, /usr/X11R6/lib
   
2. **X11 API**:
   - `XOpenDisplay` - Connect to X server
   - `XInternAtom` - Get atom identifiers
   - `XSetSelectionOwner` - Claim clipboard ownership
   - `XConvertSelection` - Request clipboard data
   - `XGetWindowProperty` - Read clipboard data
   - `XChangeProperty` - Provide clipboard data

3. **Supported formats**:
   - Text: UTF8_STRING
   - Images: image/png

4. **Requirements**:
   - libX11 must be installed
   - X11 display must be available (DISPLAY environment variable)
   - For headless: use Xvfb virtual framebuffer
   - **CGO_ENABLED=0** - No cgo required (pure Go)

5. **Thread safety**:
   - Use `runtime.LockOSThread()` for thread-sensitive operations

### macOS Implementation (Complete)

Uses purego's objc package to call Objective-C runtime and AppKit framework:

1. **Objective-C integration via purego/objc package**:
   - `objc.GetClass()` - Get class by name
   - `objc.RegisterName()` - Register method selector
   - `objc.ID.Send()` - Call Objective-C methods
   - `objc.Send[T]()` - Call methods with typed return values

2. **Dynamic library loading**:
   - AppKit framework for NSPasteboard
   - objc package auto-loads Objective-C runtime

3. **NSPasteboard API**:
   - `[NSPasteboard generalPasteboard]` - Get clipboard
   - `[pasteboard dataForType:]` - Read data
   - `[pasteboard setData:forType:]` - Write data
   - `[pasteboard changeCount]` - Monitor changes

4. **Key patterns**:
   ```go
   // Get class
   class := objc.GetClass("NSPasteboard")
   
   // Register selector
   sel := objc.RegisterName("generalPasteboard")
   
   // Call method (returns ID)
   result := objc.ID(class).Send(sel)
   
   // Call method with typed return
   count := objc.Send[int64](obj, sel)
   
   // Call method with arguments
   data := obj.Send(sel, arg1, arg2)
   ```

5. **Thread safety**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aymanbagabas/go-nativeclipboard](https://github.com/aymanbagabas/go-nativeclipboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
