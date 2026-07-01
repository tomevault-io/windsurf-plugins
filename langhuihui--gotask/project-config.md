---
trigger: always_on
description: GoTask is a sophisticated asynchronous task management framework for Go that implements an "Everything is a Task" philosophy. It provides OS-like task manager capabilities with precise control over different granularity levels of system components.
---

# Claude AI Rules for GoTask Project

## Project Understanding
GoTask is a sophisticated asynchronous task management framework for Go that implements an "Everything is a Task" philosophy. It provides OS-like task manager capabilities with precise control over different granularity levels of system components.

## Core Architectural Principles

### 1. Single Goroutine Event Loop (Fundamental Rule)
- **CRITICAL**: All child tasks execute sequentially in the parent task's goroutine
- This eliminates race conditions and ensures predictable execution
- Never create goroutines directly in task implementations
- Use `parent.AddTask(child)` instead of `go func()`

### 2. Task Hierarchy and Lifecycle
- Tasks form a tree structure with parent-child relationships
- Each task has a unique ID for tracking and management
- Use `RootManager` as the root task manager for signal handling
- Implement proper `Start()`, `Run()`, and `Dispose()` methods

### 3. Resource Management Philosophy
- Framework handles cascading disposal automatically
- Use `OnStop()` and `OnDispose()` hooks for cleanup
- Implement graceful shutdown patterns
- Never call `Start()` directly - it must be called by the parent task

## Task Type System

### task.Task (Base Task)
- Use for simple, single-purpose tasks
- Basic task implementation with lifecycle methods

### task.Job (Container Task)
- Can contain child tasks
- Ends when all child tasks complete
- Use for task coordinators and containers

### task.Work (Persistent Task)
- Similar to Job but continues after children complete
- Use for background workers and long-running services

### task.TickTask (Periodic Task)
- Executes at regular intervals
- Implement `GetTickInterval() time.Duration`
- Use for timers, heartbeats, cleanup tasks

### task.ChannelTask (Event-Driven Task)
- Custom signal-based tasks
- Override `GetSignal()` method
- Use for event-driven and reactive tasks

## Implementation Patterns

### Standard Task Template
```go
type MyTask struct {
    task.Task  // Choose appropriate type
    // Task-specific fields
    Name        string
    Config      map[string]interface{}
    Resources   []Resource
}

func (t *MyTask) Start() error {
    // Resource initialization
    t.Info("Task starting", "name", t.Name, "taskId", t.GetID())
    
    // Add resource dependencies
    t.Using(t.Resources...)
    
    // Set cleanup hooks
    t.OnStop(func() {
        // Immediate cleanup
    })
    
    return nil
}

func (t *MyTask) Run() error {
    // Main task logic (blocking)
    for !t.IsStopped() {
        // Do work
        if err := t.doWork(); err != nil {
            return err
        }
    }
    return t.StopReason()
}

func (t *MyTask) Dispose() {
    // Resource cleanup
    t.Info("Task disposing", "name", t.Name)
    for _, resource := range t.Resources {
        resource.Close()
    }
}
```

### RootManager Setup
```go
type TaskManager = task.RootManager[uint32, *MyTask]

func main() {
    // Create root manager
    root := &TaskManager{}
    root.Init()
    
    // Add tasks
    myTask := &MyTask{Name: "Example"}
    root.AddTask(myTask)
    
    // Wait for completion or handle signals
    myTask.WaitStopped()
    
    // Graceful shutdown
    root.Shutdown()
}
```

## Error Handling and Resilience

### Panic Management
- Default mode: `go build` (captures panics, converts to errors)
- Debug mode: `go build -tags taskpanic` (panics throw directly)
- Always implement proper error handling in `Run()` methods

### Retry Configuration
```go
func (t *MyTask) Start() error {
    // Configure retry strategy
    t.SetRetry(3, 5*time.Second)  // maxRetry, retryInterval
    
    // Retry logic:
    // - Start failures: retry Start() until success
    // - Run/Go failures: call Dispose(), then retry Start()
    // - Stop conditions: ErrStopByUser, ErrExit, ErrTaskComplete
    return nil
}
```

### Resource Management
```go
func (t *MyTask) Start() error {
    // Add resource dependencies
    t.Using(database, cache, httpClient)
    
    // OnStop: 用于关闭阻塞性资源（如端口监听、网络连接）
    t.OnStop(func() {
        // 关闭阻塞性资源，如 server.Close(), conn.Close()
        server.Close()
        database.Close()
    })
    
    // OnDispose: 用于清理非阻塞性资源
    t.OnDispose(func() {
        // 清理其他资源，如缓存、文件句柄等
        cache.Flush()
    })
    
    return nil
}
```

### Task Execution Model
- **Sequential Execution**: 子任务在父任务协程中顺序执行
- **Blocking Behavior**: 如果子任务的Start或Run长时间运行，会阻塞其他子任务
- **Use Cases**: 适合单个子任务重试、子任务排队执行
- **Not Suitable**: 不适合多个子任务并行处理的场景
- **Stop Method**: Stop()不能传入nil，必须提供停止原因

## Common Use Cases and Patterns

### 1. Network Service Management
```go
type HTTPService struct {
    task.Job
    Port     int
    Server   *http.Server
    Handlers map[string]http.HandlerFunc
}

func (h *HTTPService) Start() error {
    h.Server = &http.Server{
        Addr:    fmt.Sprintf(":%d", h.Port),
        Handler: h.createMux(),
    }
    
    h.OnStop(func() {
        ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
        defer cancel()
        h.Server.Shutdown(ctx)
    })
    
    return nil

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langhuihui/gotask](https://github.com/langhuihui/gotask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
