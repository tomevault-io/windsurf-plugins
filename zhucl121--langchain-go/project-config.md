---
trigger: always_on
description: 这是一个用 Go 重写 LangChain 1.2+ 和 LangGraph 1.0+ 的项目。
---

# LangChain-Go & LangGraph-Go 重写项目 Cursor Rules

## 项目概述

这是一个用 Go 重写 LangChain 1.2+ 和 LangGraph 1.0+ 的项目。

**核心目标：**
- 实现 LangGraph 1.0+ 全部核心功能（StateGraph、Checkpointing、Human-in-the-Loop）
- 实现 LangChain 核心抽象（Runnable、ChatModel、Tools）
- 性能目标：并发性能提升 10x+，内存降低 50%+

**技术栈：**
- Go 1.22+ (需要泛型支持)
- PostgreSQL/SQLite (Checkpointing)
- 标准库优先（减少依赖）

---

## 代码规范

### 1. 文件组织

```
每个包应包含：
- interface.go: 接口定义
- types.go: 类型定义
- impl.go: 实现代码
- options.go: 选项模式
- xxx_test.go: 测试文件
- doc.go: 包文档
```

### 2. 命名规范

- **接口**: 使用名词，如 `Runnable`, `ChatModel`, `Saver`
- **实现**: 使用形容词+名词，如 `PostgresSaver`, `OpenAIChatModel`
- **函数**: 使用动词开头，如 `NewXxx`, `CreateAgent`, `ExecuteNode`
- **私有字段**: 小写开头，如 `nodes`, `edges`

### 3. 错误处理

```go
// 定义包级错误
var (
    ErrInvalidInput = errors.New("xxx: invalid input")
    ErrNotFound     = errors.New("xxx: not found")
)

// 使用 fmt.Errorf 包装错误
return fmt.Errorf("failed to execute node %s: %w", name, err)

// 错误检查
if err != nil {
    return fmt.Errorf("xxx: %w", err)
}
```

### 4. 泛型使用

```go
// 优先使用泛型提供类型安全
type Runnable[I, O any] interface {
    Invoke(ctx context.Context, input I) (O, error)
}

// 状态图使用泛型
type StateGraph[S any] struct {
    // ...
}
```

### 5. 接口设计

```go
// 接口应该小而专注
type Reader interface {
    Read(p []byte) (n int, err error)
}

// 组合而非继承
type ReadWriter interface {
    Reader
    Writer
}
```

### 6. 选项模式

```go
// 使用函数选项模式
type Option func(*Options)

func WithTimeout(d time.Duration) Option {
    return func(o *Options) {
        o.Timeout = d
    }
}

// 使用
New(WithTimeout(5*time.Second), WithRetries(3))
```

### 7. Context 使用

```go
// 所有 I/O 操作必须接受 context
func (s *Saver) Put(ctx context.Context, cp Checkpoint) error {
    // 检查取消
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    
    // 传递给下游
    return s.db.ExecContext(ctx, query, args...)
}
```

### 8. 并发安全

```go
// 使用 sync.Mutex 保护共享状态
type SafeMap struct {
    mu   sync.RWMutex
    data map[string]any
}

func (m *SafeMap) Get(key string) (any, bool) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    val, ok := m.data[key]
    return val, ok
}
```

### 9. Channel 使用

```go
// 流式输出使用 channel
func (r *Runnable) Stream(ctx context.Context, input I) (<-chan Event[O], error) {
    out := make(chan Event[O], 100) // 带缓冲
    
    go func() {
        defer close(out) // 必须关闭
        
        // 检查取消
        select {
        case <-ctx.Done():
            return
        case out <- event:
        }
    }()
    
    return out, nil
}
```

---

## 核心设计模式

### 1. Runnable 接口

```go
// 所有可执行组件都实现此接口
type Runnable[I, O any] interface {
    Invoke(ctx context.Context, input I, opts ...Option) (O, error)
    Batch(ctx context.Context, inputs []I, opts ...Option) ([]O, error)
    Stream(ctx context.Context, input I, opts ...Option) (<-chan StreamEvent[O], error)
}

// 组合方式：Pipe
func Pipe[A, B, C any](first Runnable[A, B], second Runnable[B, C]) Runnable[A, C] {
    return &sequence[A, B, C]{first, second}
}
```

### 2. StateGraph 模式

```go
// 声明式 API
graph := state.NewStateGraph[MyState]("my-graph")

graph.AddNode("agent", agentNode).
    AddNode("tools", toolsNode).
    SetEntryPoint("agent").
    AddConditionalEdges("agent", routeFn, map[string]string{
        "continue": "tools",
        "end":      state.END,
    }).
    AddEdge("tools", "agent")

compiled := graph.Compile()
```

### 3. Checkpointing 模式

```go
// 接口定义
type Saver interface {
    Put(ctx context.Context, config Config, cp Checkpoint) error
    Get(ctx context.Context, config Config) (*Checkpoint, error)
    List(ctx context.Context, config Config, opts ListOptions) ([]Checkpoint, error)
}

// 使用
graph.WithCheckpointer(postgresaver).
    WithDurability(durability.ModeSync)
```

### 4. Human-in-the-Loop 模式

```go
// 节点中触发中断
func approvalNode(ctx context.Context, state State) (State, error) {
    if state.RequiresApproval {
        hitl.TriggerInterrupt(hitl.Interrupt{
            Type:    hitl.InterruptApproval,
            Message: "请审批此操作",
        })
    }
    return state, nil
}

// 恢复执行
result, err := executor.Resume(ctx, threadID, hitl.ResumeData{
    Action: hitl.ActionApprove,
})
```

### 5. Middleware 模式

```go
// 中间件接口
type Middleware interface {
    BeforeModel(ctx context.Context, state *State) (*State, error)
    AfterModel(ctx context.Context, state *State, resp *Message) (*State, error)
}

// 中间件链
agent := agents.CreateAgent(agents.Config{
    Model: model,
    Middleware: []Middleware{
        logging.New(),
        hitl.New(hitl.Config{...}),
    },
})
```

---

## 测试规范

### 1. 测试文件命名

```
file.go       -> file_test.go
interface.go  -> interface_test.go
```

### 2. 测试函数命名

```go
// 基础测试
func TestFunctionName(t *testing.T) {}

// 表格驱动测试
func TestFunctionName_Cases(t *testing.T) {
    tests := []struct {
        name    string
        input   Input
        want    Output
        wantErr bool
    }{
        {"normal case", input1, output1, false},
        {"error case", input2, nil, true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // ...
        })
    }
}
```

### 3. Mock 使用

```go
// 使用接口进行 mock
type MockSaver struct {
    PutFunc func(ctx context.Context, config Config, cp Checkpoint) error
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhucl121/langchain-go](https://github.com/zhucl121/langchain-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
