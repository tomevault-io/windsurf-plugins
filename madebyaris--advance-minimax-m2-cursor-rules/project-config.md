---
trigger: always_on
description: Go development: error handling, interfaces, concurrency, modules, and idiomatic patterns
---


# Go Development Patterns

Idiomatic Go patterns focusing on simplicity, clarity, and concurrency.

## Go Workflow

Before changing Go code:

```text
1. Read `go.mod` and existing package structure first
2. Check the current Go version in the repo before recommending newer features
3. For new modules, use `go mod init`; do not hand-create `go.mod`
4. For new dependencies or version-sensitive work, verify current versions with the actual current date
```

### CLI-First Go Development

Prefer standard Go CLI workflows:
```bash
# Project initialization (NEVER manually create go.mod)
go mod init github.com/myorg/myproject

# Add dependencies (NEVER manually edit go.mod)
go get github.com/gin-gonic/gin@latest
go get github.com/lib/pq@v1.10.9

# Verify and tidy
go mod tidy
go mod verify

# Build and test
go build ./...
go test ./...
go vet ./...

# Format before committing
go fmt ./...
gofmt -s -w .
```

### Post-Edit Verification

After meaningful Go changes, run the smallest useful check for the task:

```bash
go build ./...
go test ./...
go vet ./...
```

Run `go mod tidy` when imports or dependencies changed rather than after every tiny edit.

### Common Go Syntax Traps (Avoid These!)

```go
// WRONG: Missing error check
result, _ := doSomething()  // Never ignore errors!

// CORRECT: Always handle errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// WRONG: Range loop variable capture
for _, item := range items {
    go func() {
        process(item)  // Bug! Captures loop variable
    }()
}

// CORRECT: Pass as parameter
for _, item := range items {
    go func(i Item) {
        process(i)
    }(item)
}

// WRONG: Nil map write
var m map[string]int
m["key"] = 1  // Panic!

// CORRECT: Initialize map
m := make(map[string]int)
m["key"] = 1
```

---

## Error Handling

### Basic Error Handling
```go
// Always check errors
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// Wrap errors with context
if err := db.Query(sql); err != nil {
    return fmt.Errorf("failed to query users: %w", err)
}
```

### Custom Error Types
```go
// Simple error type
type NotFoundError struct {
    Resource string
    ID       string
}

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("%s with id %s not found", e.Resource, e.ID)
}

// Checking error types
if errors.Is(err, sql.ErrNoRows) {
    return &NotFoundError{Resource: "user", ID: id}
}

// Type assertions
var notFound *NotFoundError
if errors.As(err, &notFound) {
    // Handle not found case
}
```

### Error Handling Patterns
```go
// Early return pattern
func processData(data []byte) error {
    if len(data) == 0 {
        return errors.New("empty data")
    }
    
    parsed, err := parse(data)
    if err != nil {
        return fmt.Errorf("parse failed: %w", err)
    }
    
    if err := validate(parsed); err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    return save(parsed)
}

// Error sentinel values
var (
    ErrNotFound     = errors.New("not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInvalidInput = errors.New("invalid input")
)
```

---

## Interfaces

### Interface Design
```go
// Small, focused interfaces
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}

// Accept interfaces, return structs
func ProcessData(r io.Reader) (*Result, error) {
    data, err := io.ReadAll(r)
    if err != nil {
        return nil, err
    }
    return &Result{Data: data}, nil
}
```

### Interface Best Practices
```go
// Define interfaces where they're used, not where implemented
// In consumer package:
type UserStore interface {
    Get(id string) (*User, error)
    Save(user *User) error
}

type UserService struct {
    store UserStore  // Accepts any implementation
}

// Implementation in another package
type PostgresUserStore struct {
    db *sql.DB
}

func (s *PostgresUserStore) Get(id string) (*User, error) { ... }
func (s *PostgresUserStore) Save(user *User) error { ... }
```

### Empty Interface and Type Assertions
```go
// Avoid interface{} / any when possible
// When necessary, use type switches
func process(v any) string {
    switch x := v.(type) {
    case string:
        return x
    case int:
        return strconv.Itoa(x)
    case fmt.Stringer:
        return x.String()
    default:
        return fmt.Sprintf("%v", v)
    }
}
```

---

## Concurrency

### Goroutines and Channels
```go
// Basic goroutine
go func() {
    result := expensiveOperation()
    resultChan <- result
}()

// Buffered channel
jobs := make(chan Job, 100)

// Wait for completion
var wg sync.WaitGroup
for i := 0; i < numWorkers; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for job := range jobs {
            process(job)
        }
    }()
}
wg.Wait()
```

### Context for Cancellation
```go
func fetchData(ctx context.Context, url string) ([]byte, error) {
    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }
    
    resp, err := http.DefaultClient.Do(req)
    if err != nil {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
