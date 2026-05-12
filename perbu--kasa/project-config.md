---
trigger: always_on
description: Handles manifest file storage with git integration. Files are stored as `<baseDir>/<context>/<namespace>/<app>/<type>.yaml`. The context is the Kubernetes cluster context name, resolved from kubeconfig at startup. The git repo lives at `baseDir`; file operations are scoped to the context subdirectory.
---

# Kasa - Kubernetes Deployment Agent

## Project Overview

Kasa is a conversational Kubernetes deployment assistant built with Go, using Google's ADK (Agent Development Kit) for LLM agent capabilities and client-go for Kubernetes interaction.

## Verify build

```bash
go build -o /dev/null
```

## Versioning

Use the `bump` CLI tool to manage versions. It updates `.version`, commits, and tags in one step.

```bash
bump -patch   # v0.3.0 → v0.3.1
bump -minor   # v0.3.0 → v0.4.0
bump -major   # v0.3.0 → v1.0.0
```

**Important:** Run `bump` **after** committing your code changes, not before. The bump creates its own commit and tag. If you bump first and then commit code, the tag will point to the version-bump commit instead of the actual changes. If that happens, delete and recreate the tag on the correct commit.

## Run non-interactively:
```
go run . -prompt "list namespaces" # Single prompt mode
go run . -debug -prompt "..."      # With debug output
```

## Configuration

- `~/.kasa/config.yaml` - All settings: Kubernetes, model, API keys, system prompt
- Run `kasa init` to create a default config file
- Environment variables (`OPENROUTER_API_KEY` or `GOOGLE_API_KEY`, `JINA_API_KEY`) override config file values
- Uses OpenRouter (OpenAI-compatible API) by default; any OpenAI-compatible provider works by setting `base_url`

## Project Structure

```
kasa/
├── main.go              # Entry point, agent setup
├── openaimodel/         # Vendored OpenAI-compatible model adapter for ADK
├── tools/               # All K8s tools (one file per tool, see tools.go for registry)
├── repl/                # Interactive REPL with plan/approval workflow
├── manifest/            # Manifest file storage with git integration
├── references/          # Embedded K8s resource documentation
└── deployments/         # Git-tracked manifest storage (created at runtime)
```

## Safe Mode (Plan/Approval Workflow)

Kasa operates in **Safe Mode** by default in interactive REPL. Mutating operations require user approval before execution.

### How It Works

1. User requests a change (e.g., "deploy nginx")
2. Agent gathers information using read-only tools
3. Agent calls `propose_plan` with description and actions
4. REPL detects the plan and displays it for review
5. User types `/approve` to approve or `/abort` to reject
6. If approved, agent executes the planned actions

### Tool Categories

Tools are classified in `tools/tools.go`:

**Read-Only (use freely):**
- list_namespaces, list_pods, get_logs, get_events, get_resource
- get_reference, check_deployment_health
- list_manifests, read_manifest, dry_run_apply (supports inline YAML validation)
- list_resources (generic, supports CRDs)

**Mutating (require plan approval):**
- delete_namespace, delete_resource, delete_manifest
- apply_resource (single tool for all resource creation/updates via YAML)
- apply_manifest, import_resource, commit_manifests

### REPL Commands

- `/approve` - Approve pending plan
- `/abort` (or `/reject`) - Reject pending plan
- `/plan` - Display pending plan again
- `/debug` - Toggle debug mode

### Key Files

- `session_state.go` - `SessionState`, `Plan`, `PlannedAction` types
- `plan_display.go` - `DisplayPlan()`, `ParsePlanFromResponse()`, `FormatExecutionPrompt()`
- `tools/propose_plan.go` - The `propose_plan` tool
- `tools/tools.go` - `IsMutating()`, `ReadOnlyTools()`, `MutatingTools()`

### Non-Interactive Mode

When using `-prompt`, safe mode is disabled (no approval workflow). The agent executes directly.

## Key Patterns

### Tool Implementation

Tools implement the `tool.Tool` interface from `google.golang.org/adk/tool`. Each tool is a struct with these methods:

```go
type MyTool struct {
    // dependencies (e.g., clientset, config)
}

func NewMyTool(...) *MyTool {
    return &MyTool{...}
}

func (t *MyTool) Name() string {
    return "my_tool"
}

func (t *MyTool) Description() string {
    return "What this tool does"
}

func (t *MyTool) IsLongRunning() bool {
    return false
}

func (t *MyTool) ProcessRequest(ctx tool.Context, req *model.LLMRequest) error {
    return addFunctionTool(req, t)
}

func (t *MyTool) Declaration() *genai.FunctionDeclaration {
    return &genai.FunctionDeclaration{
        Name:        t.Name(),
        Description: t.Description(),
        Parameters: &genai.Schema{
            Type: "object",  // NOTE: string, not genai.TypeObject
            Properties: map[string]*genai.Schema{
                "param_name": {
                    Type:        "string",
                    Description: "Parameter description",
                },
            },
            Required: []string{"param_name"},
        },
    }
}

func (t *MyTool) Run(ctx tool.Context, args any) (map[string]any, error) {
    argsMap, ok := args.(map[string]any)
    if !ok {
        return map[string]any{"error": "invalid arguments"}, nil
    }

    // Execute tool logic
    result := doSomething(argsMap["param_name"].(string))

    return map[string]any{
        "result": result,
    }, nil
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perbu/kasa](https://github.com/perbu/kasa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
