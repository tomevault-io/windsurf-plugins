---
trigger: always_on
description: GitHub Copilot Instructions for NTG Agent
---


# GitHub Copilot Instructions for NTG Agent

## AI Persona

You are an experienced Senior .NET Developer. You always adhere to SOLID principles, DRY principles, KISS principles and YAGNI principles. You always follow OWASP best practices. You always break tasks down to the smallest units and approach solving any task in a step by step manner.

## General Instructions

- Make only high confidence suggestions when reviewing code changes.
- Write code with good maintainability practices, including comments on why certain design decisions were made.
- Handle edge cases and write clear exception handling.
- For libraries or external dependencies, mention their usage and purpose in comments.

## Coding Standards & Patterns

### General Guidelines
- Use **C# 12** features including file-scoped namespaces, record types, and pattern matching
- Follow **async/await** patterns consistently
- Use **dependency injection** throughout the application
- Implement proper **error handling** with try-catch blocks and logging
- Use **nullable reference types** and handle null values appropriately
- Follow **SOLID principles** and clean architecture patterns

### Naming Conventions
- Use **PascalCase** for classes, methods, properties, and public members
- Use **camelCase** for local variables and private fields
- Use **kebab-case** for CSS classes and HTML attributes
- Prefix interfaces with 'I' (e.g., `IKnowledgeService`)
- Use descriptive names that reflect business domain concepts

### File Organization
- Place models in `Models/` folders organized by domain (e.g., `Models/Chat/`, `Models/Documents/`)
- Put DTOs in `NTG.Agent.Shared.Dtos` project organized by feature
- Store services in `Services/` folders with clear interface/implementation separation
- Place Blazor components in `Components/` with subfolder organization
- Use `_Imports.razor` files for common using statements

## Entity Framework Patterns

### DbContext Usage
- Use `AgentDbContext` as the main database context
- Follow code-first approach with migrations
- Use fluent API configuration when needed
- Implement proper foreign key relationships

### Model Conventions
```csharp
public class ExampleEntity
{
    public ExampleEntity()
    {
        CreatedAt = DateTime.UtcNow;
        UpdatedAt = DateTime.UtcNow;
    }

    public Guid Id { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
    // Other properties...
}
```

### Common Entities
- **Agent** - AI assistant configuration with instructions
- **Conversation** - Chat session container
- **ChatMessage** - Individual messages with roles (User/Assistant/System)
- **Document** - Uploaded content with metadata and folder association
- **Folder** - Hierarchical document organization
- **Tag** - Content categorization system

## Blazor Component Patterns

### Component Structure
```razor
@using NTG.Agent.Shared.Dtos.SomeNamespace
@inject SomeService SomeService
@inject ILogger<ComponentName> Logger
@rendermode InteractiveServer

<!-- Markup here -->

@code {
    [Parameter] public Guid SomeId { get; set; }
    
    private bool isLoading = true;
    private string? errorMessage;
    
    protected override async Task OnInitializedAsync()
    {
        try
        {
            isLoading = true;
            // Component logic
        }
        catch (Exception ex)
        {
            Logger.LogError(ex, "Error message");
            errorMessage = "User-friendly error message";
        }
        finally
        {
            isLoading = false;
        }
    }
}
```

### Component Guidelines
- Use `@rendermode InteractiveServer` for server-side interactivity
- Use `@rendermode InteractiveWebAssembly` for client-side components
- Implement proper loading states and error handling
- Use dependency injection for services
- Follow the established parameter naming patterns

## AI Integration Patterns

### Semantic Kernel Usage
- Register kernel with multiple LLM providers (GitHub Models, OpenAI, Azure OpenAI)
- Use plugins for extensible functionality
- Implement proper context management for conversations
- Handle AI service errors gracefully

### RAG Implementation
- Use Kernel Memory for document ingestion and retrieval
- Implement semantic search with embeddings
- Provide context-aware responses using retrieved documents
- Handle document upload and processing asynchronously

## Authentication & Authorization

### Authentication Flow
- Use shared cookie authentication across services
- Implement YARP reverse proxy for BFF pattern
- Use data protection with shared keys
- Support both authenticated and anonymous sessions

### Authorization Patterns
```csharp
[Authorize(Roles = "Admin")]
public class AdminController : ControllerBase
{
    // Admin-only endpoints
}

// In Blazor components
@attribute [Authorize(Roles = "Admin")]
```

## REST API Patterns

### Controller Structure
- Place controllers in `Controllers/` folder within the service project
- Use `[ApiController]` and `[Route("api/[controller]")]` attributes
- Implement proper dependency injection in constructor
- Use async/await patterns for all operations

### API Documentation
- Use XML documentation comments for all public methods
- Document parameters, return types, and exceptions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nashtech-garage/ntg-agent](https://github.com/nashtech-garage/ntg-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
