---
trigger: always_on
description: Framework-specific patterns for agents, tools, loops, and agentic workflows.
---

# Agent Framework Patterns

Framework-specific patterns for agents, tools, loops, and agentic workflows.

## Overview

This framework implements agentic AI patterns with Claude. Understanding these patterns is essential for building effective autonomous agents.

## Agent Patterns

### Agent Interface

All agents must implement `AgentInterface`:

```php
namespace ClaudeAgents\Contracts;

interface AgentInterface
{
    /**
     * Execute the agent with the given input.
     */
    public function run(string $input): AgentResult;
    
    /**
     * Get the agent's name.
     */
    public function getName(): string;
}
```

### Creating Agents

#### Method 1: Agent Builder (Recommended)

```php
// ✅ Correct - Fluent builder API
use ClaudeAgents\Agent;

$agent = Agent::create($client)
    ->withTool($calculatorTool)
    ->withTool($searchTool)
    ->withSystemPrompt('You are a helpful assistant.')
    ->withMemory($memory)
    ->maxIterations(10)
    ->onUpdate(function ($update) {
        echo "Progress: {$update->getType()}\n";
    });

$result = $agent->run('What is 25 * 17?');
```

#### Method 2: Specific Agent Classes

```php
// ✅ Correct - Using specific agent types
use ClaudeAgents\Agents\ReactAgent;
use ClaudeAgents\Agents\ReflectionAgent;

$reactAgent = new ReactAgent($client, [
    'tools' => [$tool1, $tool2],
    'max_iterations' => 10,
    'system' => 'You are helpful.',
]);

$reflectionAgent = new ReflectionAgent($client, [
    'max_refinements' => 3,
    'quality_threshold' => 8,
]);
```

### Agent Types and Use Cases

| Agent Type | Pattern | Use Case |
|------------|---------|----------|
| `ReactAgent` | ReAct Loop | General autonomous tasks |
| `PlanExecuteAgent` | Plan first, execute | Complex multi-step workflows |
| `ReflectionAgent` | Generate, reflect, refine | High-quality outputs |
| `HierarchicalAgent` | Master-worker | Multi-domain tasks |
| `CodeGenerationAgent` | Generate + validate | Code generation |
| `DialogAgent` | Conversational | Chat applications |
| `MakerAgent` | Extreme decomposition | Million-step tasks |

## Tool System

### Tool Interface

All tools must implement `ToolInterface`:

```php
namespace ClaudeAgents\Contracts;

interface ToolInterface
{
    public function getName(): string;
    public function getDescription(): string;
    public function getInputSchema(): array;
    public function execute(array $input): ToolResult;
}
```

### Creating Tools

#### Method 1: Fluent Tool Builder

```php
// ✅ Correct - Fluent builder
use ClaudeAgents\Tools\Tool;

$weatherTool = Tool::create('get_weather')
    ->description('Get current weather for a location')
    ->parameter('city', 'string', 'City name')
    ->parameter('units', 'string', 'celsius or fahrenheit', false)
    ->required('city')
    ->handler(function (array $input): string {
        $city = $input['city'];
        $units = $input['units'] ?? 'celsius';
        
        // Call weather API
        $data = getWeatherData($city);
        
        return json_encode([
            'city' => $city,
            'temperature' => $data['temp'],
            'units' => $units,
            'conditions' => $data['conditions'],
        ]);
    });
```

#### Method 2: Tool Class

```php
// ✅ Correct - Custom tool class
namespace App\Tools;

use ClaudeAgents\Contracts\ToolInterface;
use ClaudeAgents\Tools\ToolResult;

class WeatherTool implements ToolInterface
{
    public function getName(): string
    {
        return 'get_weather';
    }
    
    public function getDescription(): string
    {
        return 'Get current weather for a location';
    }
    
    public function getInputSchema(): array
    {
        return [
            'type' => 'object',
            'properties' => [
                'city' => [
                    'type' => 'string',
                    'description' => 'City name',
                ],
                'units' => [
                    'type' => 'string',
                    'enum' => ['celsius', 'fahrenheit'],
                    'description' => 'Temperature units',
                ],
            ],
            'required' => ['city'],
        ];
    }
    
    public function execute(array $input): ToolResult
    {
        $data = $this->fetchWeather($input['city']);
        
        return ToolResult::success(json_encode($data));
    }
    
    private function fetchWeather(string $city): array
    {
        // Implementation
    }
}
```

### Tool Best Practices

1. **Single Responsibility**: Each tool does one thing well
2. **Clear Descriptions**: Help the agent understand when to use the tool
3. **Validate Input**: Check input in the handler
4. **Error Handling**: Return descriptive errors
5. **Idempotent**: Safe to call multiple times

```php
// ✅ Good tool handler
->handler(function (array $input): string {
    // Validate
    if (empty($input['query'])) {
        throw new \InvalidArgumentException('Query is required');
    }
    
    try {
        // Execute
        $results = $this->search($input['query']);
        
        // Return clear result
        return json_encode([
            'success' => true,
            'results' => $results,
            'count' => count($results),
        ]);
    } catch (\Exception $e) {
        // Handle errors

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [claude-php/claude-php-agent](https://github.com/claude-php/claude-php-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
