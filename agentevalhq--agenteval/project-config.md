---
trigger: always_on
description: AgentEval is **the comprehensive .NET evaluation toolkit for AI agents**, built primarily for **Microsoft Agent Framework (MAF)** with **Microsoft.Extensions.AI**. What RAGAS and DeepEval do for Python, AgentEval does for .NET—plus tool chain evaluation, behavioral policies, and calibrated multi-judge scoring.
---

# AgentEval - AI Coding Agent Instructions

AgentEval is **the comprehensive .NET evaluation toolkit for AI agents**, built primarily for **Microsoft Agent Framework (MAF)** with **Microsoft.Extensions.AI**. What RAGAS and DeepEval do for Python, AgentEval does for .NET—plus tool chain evaluation, behavioral policies, and calibrated multi-judge scoring.

## Architecture Overview

```
AgentEval/
├── src/
│   ├── AgentEval.Abstractions/  # Public contracts: IMetric, IEvaluableAgent, models
│   ├── AgentEval.Core/          # Implementations: metrics, assertions, comparison, tracing
│   ├── AgentEval.DataLoaders/   # Data loaders, exporters, output formatting
│   ├── AgentEval.MAF/           # Microsoft Agent Framework integration
│   ├── AgentEval.RedTeam/       # Security scanning, attack types, compliance
│   └── AgentEval/               # Umbrella packaging project (NuGet: AgentEval)
├── tests/AgentEval.Tests/       # xUnit tests, mirrors src/ structure
└── samples/AgentEval.Samples/   # 27 runnable samples (Sample01_HelloWorld, etc.)
```

All 5 library sub-projects use `RootNamespace=AgentEval` to preserve original namespaces. Only the umbrella is `IsPackable=true`; the single NuGet package contains all 5 DLLs per TFM. The CLI has been moved to its own repository at `AgentEvalHQ/AgentEval.Cli`.

## Environment Setup

Required environment variables for samples and integration tests:
```powershell
$env:AZURE_OPENAI_ENDPOINT = "https://your-resource.openai.azure.com/"
$env:AZURE_OPENAI_API_KEY = "your-api-key"
# Optional: Secondary models for comparison
$env:AZURE_OPENAI_DEPLOYMENT = "gpt-4o"           # Primary model
$env:AZURE_OPENAI_DEPLOYMENT_2 = "gpt-4o-mini"    # Secondary model
```

## Build & Test Commands

```powershell
dotnet build                 # Build all projects
dotnet test                  # Run all tests (×3 TFMs)
dotnet run --project samples/AgentEval.Samples  # Run samples
dotnet pack src/AgentEval   # Create NuGet package
```

## Key Patterns & Conventions

### Metric Interface Hierarchy
```csharp
IMetric                    // Base: EvaluateAsync(EvaluationContext) → MetricResult
├── IRAGMetric            // RequiresContext, RequiresGroundTruth
└── IAgenticMetric        // RequiresToolUsage
```

### Metric Naming Prefixes (see docs/naming-conventions.md)
- `llm_` = LLM-evaluated (API cost) → `llm_faithfulness`, `llm_relevance`
- `code_` = Computed by code (free) → `code_tool_success`, `code_tool_efficiency`  
- `embed_` = Embedding-based ($) → `embed_answer_similarity`

### Fluent Assertion Pattern
```csharp
result.ToolUsage!.Should()
    .HaveCalledTool("FeatureTool", because: "feature planning required")
        .BeforeTool("SecurityTool")
        .WithArgument("type", "OAuth2")
    .And()
    .HaveNoErrors();

result.Performance!.Should()
    .HaveTotalDurationUnder(TimeSpan.FromSeconds(10))
    .HaveEstimatedCostUnder(0.10m);
```

### Test Naming Convention
```csharp
[Fact]
public async Task MethodName_StateUnderTest_ExpectedBehavior()
// Example: HaveCalledTool_WhenToolWasCalled_ShouldPass
```

## Trace Record & Replay Pattern

Capture agent executions for deterministic replay (no LLM calls needed):
```csharp
// RECORD: Capture execution
await using var recorder = new TraceRecordingAgent(realAgent, "weather_test");
var response = await recorder.InvokeAsync("query");
var trace = recorder.Trace;
await TraceSerializer.SaveToFileAsync(trace, "trace.json");

// REPLAY: Deterministic playback
var replayer = new TraceReplayingAgent(trace);
var replayed = await replayer.InvokeAsync("query"); // Identical response
```

Multi-turn conversations:
```csharp
await using var chatRecorder = new ChatTraceRecorder(chatAgent, "support_conv");
await chatRecorder.AddUserTurnAsync("Hello");
await chatRecorder.AddUserTurnAsync("Book a flight");
var trace = chatRecorder.ToAgentTrace();
```

Workflows:
```csharp
await using var workflowRecorder = new WorkflowTraceRecorder(workflowAgent, "workflow-name");
var result = await workflowRecorder.ExecuteWorkflowAsync("Plan trip");
await WorkflowTraceSerializer.SaveToFileAsync(workflowRecorder.Trace, "workflow.json");
```

## stochastic evaluation & Model Comparison

Handle LLM non-determinism by running tests multiple times:
```csharp
var stochasticRunner = new StochasticRunner(harness, statisticsCalculator: null, EvaluationOptions);
var result = await stochasticRunner.RunStochasticTestAsync(
    agent, testCase, 
    new StochasticOptions(Runs: 10, SuccessRateThreshold: 0.8));

result.PrintTable("Metrics"); // Shows min/max/mean statistics
```

Compare models using Agent Factory pattern:
```csharp
public interface IAgentFactory {
    string ModelId { get; }
    string ModelName { get; }
    IEvaluableAgent CreateAgent();
}

// Run same test across multiple models
foreach (var factory in factories) {
    var agent = factory.CreateAgent();
    var result = await stochasticRunner.RunStochasticTestAsync(agent, testCase, options);
    modelResults.Add((factory.ModelName, result));
}

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentEvalHQ/AgentEval](https://github.com/AgentEvalHQ/AgentEval) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
