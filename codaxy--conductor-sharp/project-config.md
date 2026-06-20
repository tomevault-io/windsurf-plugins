---
trigger: always_on
description: Comprehensive guide for using ConductorSharp library to build Conductor workflows in .NET. Use when creating task handlers, workflow definitions, configuring execution engines, scaffolding definitions, or integrating ConductorSharp into .NET projects. Covers all task types, client services, patterns package, and toolkit usage.
---


# ConductorSharp Library Guide

Complete guide for building Conductor workflows using ConductorSharp's strongly-typed DSL, task handlers, and execution engine.

## Quick Reference

### Packages
- `ConductorSharp.Client` - API client
- `ConductorSharp.Engine` - Workflow engine, builder DSL, handlers
- `ConductorSharp.Patterns` - Built-in tasks (WaitSeconds, ReadWorkflowTasks, C# Lambda, Signal Wait)
- `ConductorSharp.KafkaCancellationNotifier` - Kafka cancellation support
- `ConductorSharp.Toolkit` - CLI scaffolding tool

## Project Setup

### Adding to Existing Project

```csharp
// Install packages
dotnet add package ConductorSharp.Client
dotnet add package ConductorSharp.Engine
```

### Creating New Console Project

```bash
dotnet new console -n MyConductorApp
cd MyConductorApp
dotnet add package ConductorSharp.Client
dotnet add package ConductorSharp.Engine
```

### Basic Configuration

```csharp
using ConductorSharp.Engine.Extensions;
using Microsoft.Extensions.Hosting;

var builder = Host.CreateApplicationBuilder(args);
builder.Services
    .AddConductorSharp(baseUrl: "http://localhost:8080")
    .AddExecutionManager(
        maxConcurrentWorkers: 10,
        sleepInterval: 500,
        longPollInterval: 100,
        domain: null,
        typeof(Program).Assembly
    );

builder.Services.RegisterWorkflow<MyWorkflow>();

var host = builder.Build();
await host.RunAsync();
```

## Writing Task Handlers


```csharp
using ConductorSharp.Engine.Builders.Metadata;
using ConductorSharp.Engine;
using ConductorSharp.Engine.Util;

[OriginalName("MY_TASK_name")]
public class MyTaskHandler : TaskRequestHandler<MyTaskRequest, MyTaskResponse>
{
    private readonly ConductorSharpExecutionContext _context;
    
    public MyTaskHandler(ConductorSharpExecutionContext context)
    {
        _context = context; // Access workflow/task metadata
    }

    public override async Task<MyTaskResponse> Handle(MyTaskRequest request, CancellationToken cancellationToken)
    {
        // Access context: _context.WorkflowId, _context.TaskId, _context.CorrelationId
        return new MyTaskResponse { /* ... */ };
    }
}
```

### Request/Response Models

```csharp
public class MyTaskRequest : IRequest<MyTaskResponse>
{
    [Required]
    public string InputValue { get; set; }
}

public class MyTaskResponse
{
    public string OutputValue { get; set; }
}
```

### Registering Standalone Tasks

```csharp
services.RegisterWorkerTask<MyTaskHandler>(options =>
{
    options.OwnerEmail = "team@example.com";
    options.Description = "My task description";
});
```

## Writing Workflow Definitions

### Basic Structure

```csharp
using ConductorSharp.Engine.Builders;
using ConductorSharp.Engine.Builders.Metadata;

public class MyWorkflowInput : WorkflowInput<MyWorkflowOutput>
{
    public int CustomerId { get; set; }
}

public class MyWorkflowOutput : WorkflowOutput
{
    public string Result { get; set; }
}

[OriginalName("MY_workflow")]
[WorkflowMetadata(OwnerEmail = "team@example.com")]
public class MyWorkflow : Workflow<MyWorkflow, MyWorkflowInput, MyWorkflowOutput>
{
    public MyWorkflow(WorkflowDefinitionBuilder<MyWorkflow, MyWorkflowInput, MyWorkflowOutput> builder) 
        : base(builder) { }

    // Task properties
    public SomeTaskHandler FirstTask { get; set; }
    public AnotherTaskHandler SecondTask { get; set; }

    public override void BuildDefinition()
    {
        _builder.AddTask(wf => wf.FirstTask, wf => new SomeTaskRequest { Input = wf.WorkflowInput.CustomerId });
        _builder.AddTask(wf => wf.SecondTask, wf => new AnotherTaskRequest { Input = wf.FirstTask.Output.Result });
        
        _builder.SetOutput(wf => new MyWorkflowOutput { Result = wf.SecondTask.Output.Value });
    }
}
```

### Workflow Metadata

```csharp
[WorkflowMetadata(
    OwnerEmail = "team@example.com",
    OwnerApp = "my-app",
    Description = "Workflow description",
    FailureWorkflow = typeof(FailureHandlerWorkflow)
)]
```

### Versioning

```csharp
[Version(2)]  // Version number for sub-workflow references
public class MyWorkflow : Workflow<...> { }
```

## Task Types

### Simple Task

```csharp
public MyTaskHandler MyTask { get; set; }

_builder.AddTask(wf => wf.MyTask, wf => new MyTaskRequest { InputValue = wf.WorkflowInput.Value });
```

### Sub-Workflow Task

Sub-workflows allow referencing other workflows as tasks. Define a model class that inherits from `SubWorkflowTaskModel`:

```csharp
// Define the sub-workflow model (usually scaffolded or defined separately)
[OriginalName("CHILD_workflow")]
public class ChildWorkflow : SubWorkflowTaskModel<ChildWorkflowInput, ChildWorkflowOutput> { }

// In the parent workflow:
public ChildWorkflow ChildWorkflow { get; set; }

_builder.AddTask(wf => wf.ChildWorkflow, wf => new ChildWorkflowInput { CustomerId = wf.WorkflowInput.CustomerId });
```

### Switch Task (Conditional Branching)

The Switch task evaluates a case value and executes tasks in the matching branch:

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codaxy/conductor-sharp](https://github.com/codaxy/conductor-sharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
