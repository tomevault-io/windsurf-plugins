---
trigger: always_on
description: Generates marketing content:
---

# AI Agent Module — Agents Architecture

## Overview

The **Agents** subsystem provides reusable, injectable agent classes for common AI-driven product operations. Each agent encapsulates domain-specific logic (image analysis, description generation, categorization, etc.) while leveraging the underlying pipeline architecture.

---

## Architecture

```
Agent Layer
├── BaseAgent (abstract)
│   └── Provides common execute/executeAsync patterns
├── ImageProductAgent (concrete)
│   └── Analyzes product images → structured data
├── TextDescriptionAgent (concrete)
│   └── Generates marketing copy from specifications
├── ProductCategorizerAgent (concrete)
│   └── Auto-tags and categorizes products
└── BulkProductEnricherAgent (concrete)
    └── Batch-enriches product data

             ↓ (both inject)

AgentService (orchestrator)
├── Resolves pipeline stages
├── Executes via pipeline
├── Manages sync/async dispatch
└── Returns AgentResult

             ↓

Pipeline + Stages
└── Generic: Validate → BuildPrompt → CallAI → Parse → Log
```

---

## Core Concepts

### 1. **BaseAgent** (Abstract)

All concrete agents extend `BaseAgent` which provides:

- **`execute()`** — Synchronous execution (blocking, immediate response)
- **`executeAsync()`** — Asynchronous dispatch to queue
- Dependency injection of `AgentService`
- Abstract methods: `getDefaultSystemPrompt()`, `buildInstruction()`

```php
abstract class BaseAgent
{
    abstract protected function getDefaultSystemPrompt(): string;
    abstract protected function buildInstruction(mixed $input): string;

    public function execute(mixed $input, int $agentId, int $credentialId, array $context = []): AgentResult
    public function executeAsync(mixed $input, int $agentId, int $credentialId, array $context = []): void
}
```

### 2. **Concrete Agents**

Each agent specializes in one AI task:

#### ImageProductAgent

Analyzes product images and extracts:
- Product name, description, category
- Colors, materials, dimensions
- Price estimates, quality assessment
- Key features, use cases

```php
$agent = app(ImageProductAgent::class);
$result = $agent->analyze(
    imageSource: 'https://example.com/img.jpg',  // URL or file path
    agentId: 1,
    credentialId: 1,
);
```

#### TextDescriptionAgent

Generates marketing content:
- Optimized product names
- Short & long descriptions
- Key benefits, selling points
- SEO keywords, target audience

```php
$agent = app(TextDescriptionAgent::class);
$result = $agent->execute(
    input: ['title' => 'Nike Shoes', 'specs' => [...]],
    agentId: 2,
    credentialId: 1,
);
```

#### ProductCategorizerAgent

Assigns metadata:
- Primary & secondary categories
- Tags and attributes
- Confidence levels

```php
$agent = app(ProductCategorizerAgent::class);
$result = $agent->execute(
    input: ['name' => 'Blue Cotton Shirt'],
    agentId: 3,
    credentialId: 1,
);
```

#### BulkProductEnricherAgent

Processes multiple products:
- Batch enrichment API
- Quality scoring per product
- Missing field detection

```php
$agent = app(BulkProductEnricherAgent::class);
$result = $agent->enrichBatch(
    products: [...],
    agentId: 4,
    credentialId: 1,
);
```

---

## Dependency Injection

All agents are automatically injectable via Laravel's service container.

### In Controllers

```php
class ProductController extends Controller
{
    public function importImage(ImageProductAgent $agent)
    {
        $result = $agent->analyze($imageUrl, 1, 1);
        return response()->json($result->toArray());
    }
}
```

### In Services

```php
class ProductImportService
{
    public function __construct(
        protected ImageProductAgent $imageAgent,
        protected TextDescriptionAgent $descAgent,
    ) {}

    public function enrichProduct(string $imageUrl)
    {
        $imageData = $this->imageAgent->analyze($imageUrl, 1, 1);
        $descriptions = $this->descAgent->execute($imageData->data, 2, 1);
        // ... combine and persist
    }
}
```

### In Queue Jobs

```php
class ProcessImageJob implements ShouldQueue
{
    public function handle(ImageProductAgent $agent)
    {
        $agent->analyzeAsync($this->imageUrl, 1, 1);
    }
}
```

### In Console Commands

```php
class AnalyzeImageCommand extends Command
{
    public function handle(ImageProductAgent $agent)
    {
        $result = $agent->analyze($imageUrl, 1, 1);
        $this->info('Done: ' . $result->output);
    }
}
```

---

## Creating Custom Agents

### Step 1: Extend BaseAgent

```php
namespace Webkul\AiAgent\Agents;

class MyCustomAgent extends BaseAgent
{
    protected function getDefaultSystemPrompt(): string
    {
        return <<<'PROMPT'
            You are an expert in [DOMAIN].
            Return valid JSON with keys: [...].
            PROMPT;
    }

    protected function buildInstruction(mixed $input): string
    {
        return "Analyze: " . json_encode($input);
    }
}
```

### Step 2: Use It (Auto-Injected)

```php
// No registration needed! Laravel auto-resolves from type hint.
public function myMethod(MyCustomAgent $agent)
{
    $result = $agent->execute($input, 1, 1);
}
```

### Step 3: Call execute() or executeAsync()

```php
// Sync
$result = $agent->execute($input, agentId: 1, credentialId: 1);
if ($result->success) {
    $data = $result->data;  // Parsed JSON
}

// Async

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unopim/unopim](https://github.com/unopim/unopim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
