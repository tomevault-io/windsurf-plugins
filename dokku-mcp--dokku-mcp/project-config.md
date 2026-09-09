---
trigger: always_on
description: // GOOD: Preallocate with known capacity
---

# Go Performance Best Practices

## Memory Management

### Slice Preallocation
```go
// GOOD: Preallocate with known capacity
func processApplications(apps []Application) []ProcessedApp {
    processed := make([]ProcessedApp, 0, len(apps))
    
    for _, app := range apps {
        processed = append(processed, processApp(app))
    }
    
    return processed
}

// BAD: Progressive growth is expensive
func processApplicationsBad(apps []Application) []ProcessedApp {
    var processed []ProcessedApp // Will start at zero, require multiple reallocations
    
    for _, app := range apps {
        processed = append(processed, processApp(app))
    }
    
    return processed
}
```

### Buffer Reuse Pattern
```go
// Buffer reuse to reduce allocations
type LogProcessor struct {
    buffer    bytes.Buffer
    jsonBuf   bytes.Buffer
    mu        sync.Mutex // Protect concurrent access
}

func NewLogProcessor() *LogProcessor {
    return &LogProcessor{
        buffer:  bytes.Buffer{},
        jsonBuf: bytes.Buffer{},
    }
}

func (lp *LogProcessor) FormatLog(entry LogEntry) string {
    lp.mu.Lock()
    defer lp.mu.Unlock()
    
    lp.buffer.Reset() // Reuse buffer
    lp.buffer.WriteString(entry.Timestamp.Format(time.RFC3339))
    lp.buffer.WriteString(" [")
    lp.buffer.WriteString(entry.Level)
    lp.buffer.WriteString("] ")
    lp.buffer.WriteString(entry.Message)
    
    return lp.buffer.String()
}

// Buffer pool for high concurrency
var bufferPool = sync.Pool{
    New: func() interface{} {
        return &bytes.Buffer{}
    },
}

func FormatLogConcurrent(entry LogEntry) string {
    buf := bufferPool.Get().(*bytes.Buffer)
    defer bufferPool.Put(buf)
    
    buf.Reset()
    buf.WriteString(entry.Timestamp.Format(time.RFC3339))
    buf.WriteString(" [")
    buf.WriteString(entry.Level)
    buf.WriteString("] ")
    buf.WriteString(entry.Message)
    
    return buf.String()
}
```

### String Building Optimization
```go
// GOOD: Use strings.Builder for string construction
func buildDokkuCommand(command string, args []string, envVars map[string]string) string {
    var builder strings.Builder
    
    // Estimate capacity to avoid reallocations
    capacity := len(command) + 10 // command + spaces
    for _, arg := range args {
        capacity += len(arg) + 1 // arg + space
    }
    for k, v := range envVars {
        capacity += len(k) + len(v) + 2 // key=value + space
    }
    
    builder.Grow(capacity)
    
    // Build environment variables
    for key, value := range envVars {
        builder.WriteString(key)
        builder.WriteByte('=')
        builder.WriteString(value)
        builder.WriteByte(' ')
    }
    
    builder.WriteString(command)
    for _, arg := range args {
        builder.WriteByte(' ')
        builder.WriteString(arg)
    }
    
    return builder.String()
}

// BAD: Repeated string concatenation
func buildDokkuCommandBad(command string, args []string, envVars map[string]string) string {
    result := ""
    
    for key, value := range envVars {
        result += key + "=" + value + " " // Creates new strings each time
    }
    
    result += command
    for _, arg := range args {
        result += " " + arg // More new allocations
    }
    
    return result
}
```

## Profiling Integration

### Built-in Profiling Support
```go
//go:build debug

package main

import (
    "context"
    "log"
    "net/http"
    _ "net/http/pprof" // HTTP profiling endpoints
    "os"
    "os/signal"
    "syscall"
    "time"
)

func init() {
    // Profiling server in debug mode
    go func() {
        log.Println("Profiling server started on :6060")
        log.Println("Visit http://localhost:6060/debug/pprof/ for profiling")
        log.Println(http.ListenAndServe(":6060", nil))
    }()
}

// Example of profiling usage in application
func main() {
    // Conditional profiling configuration
    if os.Getenv("ENABLE_PROFILING") == "true" {
        go startProfilingServer()
    }
    
    // Main application logic
    startApplication()
}

func startProfilingServer() {
    mux := http.NewServeMux()
    
    // Custom profiling endpoints
    mux.HandleFunc("/debug/pprof/", http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        log.Printf("Profiling request: %s", r.URL.Path)
        http.DefaultServeMux.ServeHTTP(w, r)
    }))
    
    server := &http.Server{
        Addr:    ":6060",
        Handler: mux,
    }
    
    log.Printf("Profiling server available at http://localhost:6060/debug/pprof/")
    if err := server.ListenAndServe(); err != nil {
        log.Printf("Profiling server error: %v", err)
    }
}
```

### Benchmarking Critical Operations
```go
func BenchmarkApplicationDeploy(b *testing.B) {
    service := setupDeploymentService()
    app := createTestApplication("benchmark-app")
    
    b.ResetTimer()
    b.ReportAllocs() // Report memory allocations
    
    for i := 0; i < b.N; i++ {
        if err := service.Deploy(context.Background(), app); err != nil {
            b.Fatalf("deployment failed: %v", err)
        }
    }
}

func BenchmarkLogFormatting(b *testing.B) {
    processor := NewLogProcessor()
    entry := LogEntry{
        Timestamp: time.Now(),
        Level:     "INFO",
        Message:   "Log formatting performance test",
    }
    
    b.ResetTimer()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dokku-MCP/dokku-mcp](https://github.com/dokku-MCP/dokku-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
