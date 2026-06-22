---
trigger: always_on
description: This is the CANONICAL agent-instructions file. `CLAUDE.md` is a symlink to it,
---

<!--
  This is the CANONICAL agent-instructions file. `CLAUDE.md` is a symlink to it,
  so edit rules HERE — Claude Code reads it through `CLAUDE.md`, OpenCode reads
  `AGENTS.md` directly. Both see exactly this content.
-->

# Lesstruct — Agent Instructions

## Ramp-up — read in this order before your first task

1. **This file** — coding, testing, and doc-sync conventions.
2. **`docs/_index.md`** — the doc map ("for X, read Y").
3. **`docs/project-context.md`** — architecture, layers, and the **"Where does new code go?"** decision tree.
4. **`docs/api-reference.md`** — only if you are touching `/api/v1`.
5. **One exemplar of each layer**: a handler (`internal/api/handlers/`), a service (`internal/domain/<name>/service.go`), and a `*_test.go` under `internal/domain/`.

> The architecture, layer responsibilities, error/response conventions, and the
> decision tree live in `docs/project-context.md`. This file is the *style & process* contract; that file is the *architecture* contract. Read both.

## Code Style

- Put all private structs or functions before all public structs or functions!
- Do not ever use `interface{}`, use `any`!
- Always use constants instead of typed strings, especially to define HTTP methods! E.g., `http.MethodDelete` instead of `"DELETE"`.

- If a function has many arguments, do not put those arguments into one line, but use multiple lines instead!
```go
// Good example

authHandler := handlers.NewAuthHandler(
  authService,
  jwtManager,
  logger,
  firstLoginService,
  registrationService,
  verificationService,
  loginService,
  userRepo,
  failedLoginRepo,
  notificationRepo,
  emailService,
)
```
```go
// Bad example

authHandler := handlers.NewAuthHandler(authService, jwtManager, logger, firstLoginService, registrationService, verificationService, loginService, userRepo, failedLoginRepo, notificationRepo, emailService)
```

## Structs & Constructors

- Treat `struct` like objects if they have function receivers (methods)! So, put constructors (functions that start with `New` in general) after all methods. Put function receivers (methods) right after their `struct`.

```go
// Good example

type Name struct {
    First string
    Last  string
}

func (n Name) Full() string {
    if n.Last == "" {
        return n.First
    }
    return fmt.Sprintf("%s %s", n.First, n.Last)
}

func NewName(first, last string) (Name, error) {
    if first == "" {
        return Name{}, errors.New("first name cannot be empty")
    }
    return Name{
        First: first,
        Last: last,
    }, nil
}
```

```go
// Bad example

type Name struct {
    First string
    Last  string
}

func NewName(first, last string) (Name, error) {
    if first == "" {
        return Name{}, errors.New("first name cannot be empty")
    }
    return Name{
        First: first,
        Last: last,
    }, nil
}

func (n Name) Full() string {
    if n.Last == "" {
        return n.First
    }
    return fmt.Sprintf("%s %s", n.First, n.Last)
}
```

## Logging & Error Handling

- Do not ever use `panic()`!
- Use `log.Fatalf()` or `log.Panicf()` only in the `main.go`!

## Mocks

- Always use `make mock` to generate mock files!
- Always use `github.com/stretchr/testify/mock` for writing mocks.

## Testing

- Do not test your works by `go build` or `go run`, but by `go test` instead!
- Always use packages that end with `_test` for all test files. Do not test private functions directly, but through the public ones.
- Always use `github.com/stretchr/testify` for writing unit tests.
- Make sure the domain layer ( @internal/domain/ ) has 100% test coverage! Remove unreachable code or skip errors using `_` variables if needed.
- Ensure your works pass `make lint`, `make test`, and `make vulncheck`!

### Before Touching Any Test File
- **Read the entire test file first.** Never edit based on partial context.
- Identify the exact struct field names in the test table before writing anything.
- If the function under test uses factory functions, read their signatures before using them.

### Never Use These to Edit `.go` Files
- **Never use `sed`, `awk`, or any regex-based shell command to edit `.go` files.** Always use full-block rewrites or exact string replacement.

### Test File Structure
- All tests must be table-driven using a named struct slice.
- Always use `tt` as the loop variable and `t.Run(tt.name, ...)` for subtests.
- Always include a `name string` as the first field in the test struct.
- Always include `wantErr bool` if the function under test returns an error.

```go
// Good example

func TestSomething(t *testing.T) {
    tests := []struct {
        name     string
        input    InputType
        expected ExpectedType
        wantErr  bool
    }{
        {
            name:     "success - valid input",
            input:    NewInputType(...),
            expected: NewExpectedType(...),
            wantErr:  false,
        },
        {
            name:    "error - invalid input",
            input:   NewInputType(...),
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := FunctionUnderTest(tt.input)
            if tt.wantErr {
                require.Error(t, err)
                return
            }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aristorinjuang/lesstruct](https://github.com/aristorinjuang/lesstruct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-22 -->
