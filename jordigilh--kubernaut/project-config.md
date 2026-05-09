---
trigger: always_on
description: Technical implementation standards: Go patterns, AI/ML integration, and system architecture
---

# Technical Implementation Standards

## 🔧 **Go Coding Standards**

### Code Organization
- **Business Names**: Use descriptive names reflecting business domain (`EffectivenessAssessor`, `WorkflowEngine`)
- **Business Requirements**: Every component MUST serve documented business requirement (BR-[CATEGORY]-[NUMBER])
- **Package Cohesion**: Group related functionality following DDD principles
- **Interface Design**: Implement interfaces over concrete types for testability
- **Unique Names**: Avoid duplicating structure names - use unique, business-aligned identifiers

### Error Handling
```go
// Always wrap errors with context
return fmt.Errorf("operation description: %w", err)

// Use structured error types
return &internal.BusinessError{
    Operation: "workflow execution",
    Cause:     err,
    Context:   map[string]interface{}{"workflowID": id},
}

// Log with structured fields
logger.WithError(err).WithField("operation", "validate").Error("validation failed")
```

### Context and Cancellation
```go
// Always accept context as first parameter
func ProcessWorkflow(ctx context.Context, workflow *Workflow) error

// Respect context cancellation
for {
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
        // Continue processing
    }
}

// Use context for request-scoped values
traceID := ctx.Value("traceID").(string)
```

### Type System Guidelines
- **MANDATORY**: Avoid using `any` or `interface{}` unless absolutely necessary
- **ALWAYS**: Use structured field values with specific types
- **AVOID**: Local type definitions to resolve import cycles
- **USE**: Shared types from `pkg/shared/types/` package instead
- **PREFER**: Strongly-typed interfaces that reflect business domain concepts

## 🤖 **AI/ML Integration Architecture**

### Supported AI Providers
| Provider | Use Case | Integration Path |
|----------|----------|------------------|
| **HolmesGPT** | Primary AI service | `pkg/ai/holmesgpt/client.go` |
| **OpenAI** | GPT-3.5, GPT-4 models | Direct API integration |
| **Anthropic** | Claude models | API client |
| **Azure OpenAI** | Enterprise deployment | Azure SDK |
| **AWS Bedrock** | Amazon AI service | AWS SDK |
| **Ollama** | Local LLM deployment | Local API |
| **Ramalama** | Local model serving | Local API |

### HolmesGPT Integration Pattern
```go
// Use the unified HolmesGPT client
holmesClient := holmesgpt.NewClient(config.HolmesGPT)
response, err := holmesClient.AnalyzeAlert(ctx, alertData)
if err != nil {
    return fmt.Errorf("HolmesGPT analysis failed: %w", err)
}
```

### AI Response Processing Pipeline
1. **Structure Validation**: Ensure response matches expected schema
2. **Confidence Scoring**: Evaluate AI recommendation confidence
3. **Safety Validation**: Check recommendations against safety policies
4. **Business Rule Validation**: Ensure recommendations align with business logic

### Confidence Thresholds
```go
type ConfidenceLevel struct {
    High   float64 // >0.8 - Execute automatically
    Medium float64 // 0.5-0.8 - Require approval
    Low    float64 // <0.5 - Log only, no action
}
```

### AI Safety and Reliability
```go
// Circuit breaker for AI service calls
breaker := circuitbreaker.New(&Config{
    Timeout:     30 * time.Second,
    MaxRequests: 100,
    Interval:    60 * time.Second,
})
```

#### Fallback Strategies
1. **Primary**: HolmesGPT with full context
2. **Secondary**: Direct LLM provider with reduced context
3. **Fallback**: Rule-based decision making
4. **Emergency**: Safe default actions only

## 🗄️ **System Architecture Patterns**

### Database Access
```go
// PostgreSQL with connection pooling
db := postgresql.NewPool(config.Database)

// Prepared statements
stmt, err := db.Prepare("SELECT * FROM workflows WHERE id = $1")

// Transaction management
tx, err := db.Begin()
defer tx.Rollback() // Will be ignored if committed
// ... operations
tx.Commit()

// Vector database operations
vectorDB := vector.NewClient(config.VectorDB)
embeddings, err := vectorDB.SimilaritySearch(ctx, query, limit)
```

### Kubernetes Client Patterns
```go
// Use shared client
k8sClient := k8s.NewClient(config.Kubernetes)
defer k8sClient.Close()

// Safety checks before destructive operations
if err := k8sClient.ValidateAccess(ctx, namespace, resource); err != nil {
    return fmt.Errorf("insufficient permissions: %w", err)
}

// Dry-run validation
if err := k8sClient.DryRun(ctx, operation); err != nil {
    return fmt.Errorf("dry-run failed: %w", err)
}
```

### Concurrency Patterns
```go
// Worker pools with resource limits
pool := workerpool.New(maxWorkers)

// Circuit breakers for external services
breaker := circuitbreaker.New(failureThreshold)

// sync.Once for expensive initialization
var once sync.Once
once.Do(func() { initializeExpensiveResource() })

// Prefer channels over shared memory
results := make(chan ProcessingResult, bufferSize)
```

## 🧠 **Workflow Engine AI Integration**

### Intelligent Workflow Builder
**Location**: `pkg/workflow/engine/intelligent_workflow_builder_impl.go`
- AI-generated multi-step remediation workflows
- Dynamic template generation based on alert patterns
- Learning from historical workflow effectiveness

### AI Condition Evaluator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordigilh/kubernaut](https://github.com/jordigilh/kubernaut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
