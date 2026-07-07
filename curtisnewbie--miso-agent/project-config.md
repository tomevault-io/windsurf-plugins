---
trigger: always_on
description: `miso-agent` is a **pure Go library** (`github.com/curtisnewbie/miso-agent`) for building LLM-powered ReAct agents. It is built on top of the [Eino](https://github.com/cloudwego/eino) graph orchestration framework and the author's own [miso](https://github.com/curtisnewbie/miso) microservice framework. There is no `main.go`; this project is consumed as a dependency.
---

# miso-agent AGENTS.md

## Project Overview

`miso-agent` is a **pure Go library** (`github.com/curtisnewbie/miso-agent`) for building LLM-powered ReAct agents. It is built on top of the [Eino](https://github.com/cloudwego/eino) graph orchestration framework and the author's own [miso](https://github.com/curtisnewbie/miso) microservice framework. There is no `main.go`; this project is consumed as a dependency.

---

## Build / Lint / Test Commands

```sh
# Build all packages (must pass with zero errors after any code change)
go build ./...

# Static analysis
go vet ./...

# Format code (ALWAYS run before committing)
go fmt ./...

# Run a single test by name (preferred approach to avoid timeout from integration tests)
go test -run TestFunctionName -count=1 -timeout 30s ./package/...

# Examples
go test -run TestAgent_shouldEvictToolResult -count=1 -timeout 30s ./agentloop/...
go test -run TestBuiltinTools_ReadFile -count=1 -timeout 30s ./agentloop/...
go test -run TestTokenizer_PruneMessagesToTokenLimit -count=1 -timeout 30s ./agentloop/...

# Run all unit tests in a specific package
go test -count=1 -timeout 60s ./agentloop/...

# WARNING: `go test ./...` may hang indefinitely — some tests in testapi/ require
# live external services (Redis, LLM APIs). Always use -timeout or run by package.
```

---

## Package Structure

| Package | Path | Purpose |
|---------|------|---------|
| `agentloop` | `agentloop/` | Core ReAct agent loop: tool calling, skills system, token management, prompt building, file-backed storage |
| `agents` | `agents/` | Pre-built specialized agents: RuleMatcher, MaterialExtract, ExecutiveSummaryWriter, MemorySummarizer, DeepResearchClarifier, plus model factory |
| `prebuilt` | `prebuilt/` | Higher-level ready-to-use agents: CsvFormatAgent, ClassificationAgent, FactCheck, AccuracyCheck, RelevanceCheck, CategoryAnalyze, ContextualRetrieval, plus shared Eval wrappers |
| `tools` | `tools/` | Reusable `agentloop.Tool` implementations: TavilySearch, DifyRetrieval — pass via `AgentConfig.Tools` |
| `graph` | `graph/` | Eino graph wrappers: compilation, invocation with trace callbacks, Mermaid diagram visualization, `GenericOps` config |
| `memory` | `memory/` | Conversational memory: short-term (Redis-backed conversation list) and long-term (Redis-backed summary with auto-compaction) |
| `agentapi` | `agentapi/` | External API integrations: Tavily Deep Research and Background Check wrappers |
| `testapi` | `testapi/` | Smoke tests: exercises graph compilation for all agents to verify they build successfully |

---

## Code Style Guidelines

### Import Organization

Imports are grouped into **3 blocks** separated by blank lines, in this order:

```go
import (
    // 1. Standard library
    "context"
    "fmt"
    "strings"
    "sync"

    // 2. Third-party dependencies
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/schema"
    "gopkg.in/yaml.v2"

    // 3. Internal packages + miso framework (treated as one group)
    "github.com/curtisnewbie/miso-agent/graph"
    "github.com/curtisnewbie/miso/errs"
    "github.com/curtisnewbie/miso/flow"
    "github.com/curtisnewbie/miso/util/strutil"
)
```

### Naming Conventions

| Category | Convention | Examples |
|----------|-----------|---------|
| Structs (exported) | `PascalCase`, noun-based | `Agent`, `AgentConfig`, `ToolRegistry`, `FileStore` |
| Structs (unexported) | `camelCase` | `ctxKey`, `agentLoopState`, `taskInput`, `openAiModelConfig` |
| Interfaces | `PascalCase`, behavior-named, no `I` prefix | `FileStore`, `Tool`, `SelfInvokeTool` |
| Constructors | `NewXxx(...)` | `NewAgent`, `NewToolRegistry`, `NewMemFileStore` |
| Functional options | `WithXxx(...)` returning `func(o *Config)` | `WithTemperature`, `WithMaxToken`, `WithSkills` |
| Methods | verb-first | `Execute`, `Build`, `Load`, `Register`, `AddTodo` |
| Exported variables | `PascalCase` | `DeepseekBaseURL`, `BasePrompt` |
| Unexported variables | `camelCase` | `agentCtxKey`, `finishToolName`, `modelMaxToken` |
| Constants | `PascalCase` (exported), `camelCase` (unexported) | `BasePrompt`, `maxToken32k` |
| Files | `snake_case` | `skill_loader.go`, `tools_builtin.go`, `tools_artifact.go` |
| Input/Output structs | `XxxInput` / `XxxOutput` | `RuleMatcherInput`, `MaterialExtractOutput` |
| Args structs for tools | `XxxArgs` | `ReadFileArgs`, `WriteFileArgs` |

### Error Handling

Use the `github.com/curtisnewbie/miso/errs` package. **Do not use `fmt.Errorf`** (except when using `%w` in stdlib-only contexts).

```go
// Create a new formatted error
return errs.NewErrf("file not found: %s", path)
return errs.NewErrf("task cannot be empty")

// Wrap an existing error with context (most common)
return nil, errs.Wrapf(err, "failed to load skills from %s", source)
return nil, errs.Wrapf(err, "failed to initialize tokenizer for model %s", config.Model)

// Simple wrap without message
return nil, errs.Wrap(err)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CurtisNewbie/miso-agent](https://github.com/CurtisNewbie/miso-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
