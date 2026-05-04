---
trigger: always_on
description: Vizra ADK is a comprehensive Laravel package for building intelligent AI agents with tool usage, memory persistence, and workflow capabilities. This package integrates with multiple LLM providers (OpenAI, Anthropic, Google Gemini) through Prism PHP.
---

# Vizra ADK - Laravel AI Agent Development Kit

## Project Overview
Vizra ADK is a comprehensive Laravel package for building intelligent AI agents with tool usage, memory persistence, and workflow capabilities. This package integrates with multiple LLM providers (OpenAI, Anthropic, Google Gemini) through Prism PHP.

**Package Name**: vizra/vizra-adk  
**Version**: 0.0.17  
**PHP**: ^8.2  
**Laravel**: ^11.0 | ^12.0  
**License**: MIT

## Key Concepts

### Agents
- **BaseLlmAgent**: Core agent class that interfaces with LLMs
- **BaseWorkflowAgent**: Agent for orchestrating complex workflows
- Auto-discovery: Agents are automatically discovered without manual registration
- Support for sub-agent delegation and task distribution

### Tools
- Implement `ToolInterface` to create custom tools
- Tools extend agent capabilities (database, API, external services)
- MCP (Model Context Protocol) support for external tool servers
- Built-in tools: MemoryTool, VectorMemoryTool, DelegateToSubAgentTool

### Memory System
- Persistent conversation memory across sessions
- Vector memory for RAG (Retrieval Augmented Generation)
- Multiple embedding providers: OpenAI, Cohere, Gemini, Ollama
- Meilisearch integration for vector storage

### Workflows
- SequentialWorkflow: Execute agents in order
- ParallelWorkflow: Run multiple agents simultaneously
- ConditionalWorkflow: Branch based on conditions
- LoopWorkflow: Iterate over data or conditions

## Directory Structure

```
vizra-adk/
├── src/
│   ├── Agents/           # Core agent implementations
│   ├── Tools/            # Tool implementations and MCP wrapper
│   ├── Memory/           # Memory management
│   ├── Services/         # Core services (Manager, Registry, Tracer)
│   ├── Models/           # Eloquent models
│   ├── Console/Commands/ # Artisan commands
│   ├── Evaluations/      # Testing and evaluation framework
│   ├── Events/           # Laravel events
│   ├── Http/             # Controllers and API endpoints
│   ├── Livewire/         # Dashboard components
│   └── Providers/        # Service providers and embeddings
├── tests/
│   ├── Unit/             # Unit tests
│   ├── Feature/          # Feature tests
│   └── Integration/      # Integration tests
├── config/               # Package configuration
├── database/migrations/  # Database migrations
├── resources/views/      # Blade templates and Livewire views
└── examples/             # Example agents and tools
```

## Testing

### Test Framework
- **Pest PHP**: Modern testing framework
- **Orchestra Testbench**: Laravel package testing

### Running Tests
```bash
# Run all tests
./vendor/bin/pest

# Run with coverage
./vendor/bin/pest --coverage

# Run specific test file
./vendor/bin/pest tests/Unit/Agents/BaseLlmAgentTest.php

# Run tests in parallel
./vendor/bin/pest --parallel
```

### Test Structure
- Unit tests: Isolated component testing
- Feature tests: End-to-end functionality
- Integration tests: Cross-component interactions

## Development Commands

### Artisan Commands
```bash
# Install package (migrations, config)
php artisan vizra:install

# Create new agent
php artisan vizra:make:agent MyAgent

# Create new tool
php artisan vizra:make:tool MyTool

# Create evaluation
php artisan vizra:make:eval MyEvaluation

# Create assertion
php artisan vizra:make:assertion MyAssertion

# Interactive chat with agent
php artisan vizra:chat agent_name

# List discovered agents
php artisan vizra:agents

# Run evaluations
php artisan vizra:eval:run

# Start web dashboard
php artisan vizra:dashboard

# View agent trace
php artisan vizra:trace {traceId}

# Clean old traces
php artisan vizra:trace:cleanup

# Manage prompt versions
php artisan vizra:prompts

# MCP server management
php artisan vizra:mcp:list

# Vector memory operations
php artisan vizra:vector:store --file={file}
php artisan vizra:vector:search {query}
php artisan vizra:vector:stats
```

## Code Conventions

### Naming Conventions
- Agents: `{Purpose}Agent` extends `BaseLlmAgent`
- Tools: `{Action}Tool` implements `ToolInterface`
- Evaluations: `{Scenario}Evaluation` extends `BaseEvaluation`
- Assertions: `{Check}Assertion` extends `BaseAssertion`

### Namespace Structure
```php
Vizra\VizraADK\Agents\      # Agent classes
Vizra\VizraADK\Tools\       # Tool implementations
Vizra\VizraADK\Services\    # Core services
Vizra\VizraADK\Models\      # Eloquent models
Vizra\VizraADK\Evaluations\ # Evaluation framework
```

### Agent Definition Pattern
```php
class MyAgent extends BaseLlmAgent
{
    protected string $name = 'my_agent';
    protected string $description = 'Agent purpose';
    protected string $instructions = 'Detailed instructions';
    protected string $model = 'gpt-4o';
    protected array $tools = [
        MyTool::class,
    ];
}
```

### Tool Implementation Pattern
```php
class MyTool implements ToolInterface
{
    public function definition(): array
    {
        return [
            'name' => 'tool_name',
            'description' => 'What this tool does',
            'parameters' => [
                'type' => 'object',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vizra-ai/vizra-adk](https://github.com/vizra-ai/vizra-adk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
