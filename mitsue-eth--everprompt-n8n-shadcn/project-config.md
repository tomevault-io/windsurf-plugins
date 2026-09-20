---
trigger: always_on
description: n8n workflow JSON parser and prompt extraction strategy
---


# n8n Workflow Parser & Prompt Extraction

## Core Concept

**Simple User Flow**: User completes n8n workflow → uploads JSON → EverPrompt extracts prompts → creates organized collection

## Workflow Parser Architecture

### 1. **JSON Parser Interface**

```typescript
interface N8nWorkflowParser {
  parseWorkflow(json: string): ParsedWorkflow;
  extractPrompts(workflow: ParsedWorkflow): ExtractedPrompt[];
  createCollection(
    prompts: ExtractedPrompt[],
    workflowName: string
  ): PromptCollection;
}

interface ParsedWorkflow {
  name: string;
  nodes: WorkflowNode[];
  connections: WorkflowConnection[];
  metadata: WorkflowMetadata;
}

interface WorkflowNode {
  id: string;
  name: string;
  type: string;
  position: [number, number];
  parameters: Record<string, any>;
  credentials?: Record<string, any>;
}
```

### 2. **Prompt Extraction Logic**

```typescript
interface ExtractedPrompt {
  nodeId: string;
  nodeName: string;
  nodeType: string;
  promptType: "system" | "user" | "template";
  content: string;
  variables: string[];
  position: [number, number];
  metadata: {
    model?: string;
    temperature?: number;
    maxTokens?: number;
    credentials?: string;
  };
}

class N8nPromptExtractor {
  extractFromNode(node: WorkflowNode): ExtractedPrompt[] {
    const prompts: ExtractedPrompt[] = [];

    // Extract from different node types
    switch (node.type) {
      case "n8n-nodes-base.perplexity":
        prompts.push(...this.extractPerplexityPrompts(node));
        break;
      case "@n8n/n8n-nodes-langchain.openAi":
        prompts.push(...this.extractOpenAIPrompts(node));
        break;
      case "n8n-nodes-base.chatGpt":
        prompts.push(...this.extractChatGPTPrompts(node));
        break;
      // Add more node types as needed
    }

    return prompts;
  }
}
```

## Supported Node Types

### 1. **Perplexity Nodes**

```typescript
extractPerplexityPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.message) {
    params.messages.message.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'perplexity',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.model,
          temperature: params.temperature,
          searchRecency: params.options?.searchRecency
        }
      });
    });
  }

  return prompts;
}
```

### 2. **OpenAI/LangChain Nodes**

```typescript
extractOpenAIPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.values) {
    params.messages.values.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'openai',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.modelId?.value,
          temperature: params.temperature,
          maxTokens: params.maxTokens
        }
      });
    });
  }

  return prompts;
}
```

### 3. **ChatGPT Nodes**

```typescript
extractChatGPTPrompts(node: WorkflowNode): ExtractedPrompt[] {
  const prompts: ExtractedPrompt[] = [];
  const params = node.parameters;

  if (params.messages?.message) {
    params.messages.message.forEach((msg: any, index: number) => {
      prompts.push({
        nodeId: node.id,
        nodeName: node.name,
        nodeType: 'chatgpt',
        promptType: msg.role === 'system' ? 'system' : 'user',
        content: msg.content,
        variables: this.extractVariables(msg.content),
        position: node.position,
        metadata: {
          model: params.model,
          temperature: params.temperature,
          maxTokens: params.maxTokens
        }
      });
    });
  }

  return prompts;
}
```

## Variable Extraction

### 1. **n8n Expression Parser**

```typescript
extractVariables(content: string): string[] {
  const variables: string[] = [];

  // Extract n8n expressions like {{ $json.field }}
  const expressionRegex = /\{\{\s*\$([^}]+)\s*\}\}/g;
  let match;

  while ((match = expressionRegex.exec(content)) !== null) {
    variables.push(match[1].trim());
  }

  // Extract function calls like =function()
  const functionRegex = /=\s*([a-zA-Z_][a-zA-Z0-9_]*)\s*\(/g;
  while ((match = functionRegex.exec(content)) !== null) {
    variables.push(match[1]);
  }

  return [...new Set(variables)]; // Remove duplicates
}
```

### 2. **Variable Types**

```typescript
interface N8nVariable {
  name: string;
  type: "json" | "function" | "constant";
  path?: string; // For JSON variables like $json.field
  functionName?: string; // For function calls
  description?: string;
}
```

## Collection Creation

### 1. **Workflow Collection**

```typescript
interface WorkflowCollection {
  id: string;
  name: string;
  description: string;
  workflowId: string;
  workflowName: string;
  prompts: ExtractedPrompt[];
  metadata: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
