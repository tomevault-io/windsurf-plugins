---
trigger: always_on
description: I don't need layers of abstraction. Base LLMs are very capable. Feather lightly defines agents with tools that auto-execute.
---

# FEATHER DOCS - A lightweight agent framework

I don't need layers of abstraction. Base LLMs are very capable. Feather lightly defines agents with tools that auto-execute.

Chaining agents together with Feather looks like this:

```typescript
const timelineData = twitterAgent.run("Get 50 tweets from my AI list and summarize it for me")
const videoScript = videoScriptAgent.run('Create me a video script based on todays AI news:' + timelineData.output)
```

## CREATING AN AGENT

Creating an agent is easy:

```typescript
const internetAgent = new FeatherAgent({
    model: "deepseek/deepseek-chat",
    systemPrompt: "You are a helpful assistant that can browse the internet", 
    tools: [internetTool],
})
```

Running an agent is easier:

```typescript
const result = internetAgent.run("What's the latest quantum chip that dropped? How does it advance AI?")
logger.info("Internet Agent said:", result.output)
```

The output comes from the .output property of the agent run.
Agent run has the following properties:

```typescript
interface AgentRunResult<TOutput> {
  // Whether the agent run completed successfully
  success: boolean;

  // The main output from the agent run
  // Type depends on agent configuration:
  // - string for normal text output
  // - TOutput for structured output (if structuredOutputSchema is used)
  // - Record<string, any> for JSON responses
  output: TOutput;

  // Only present if there was an error during the run
  error?: string;

  // Only present if autoExecuteTools is false and the agent wants to use tools
  functionCalls?: Array<{
    functionName: string;    // Name of the tool/function to call
    functionArgs: any;       // Arguments for the tool/function
  }>;
}
```

### FeatherAgent Parameters

Required:
- `systemPrompt` (string) - Core instructions that define the agent's behavior

Optional:
- `model` (string) - LLM model to use (defaults to "openai/gpt-4o")
- `agentId` (string) - Unique ID for the agent (auto-generates if not provided) 
- `dynamicVariables` (object) - Functions that return strings, executed on each .run() call
- `tools` (ToolDefinition[]) - Tools the agent can use (cannot use with structuredOutputSchema)
- `autoExecuteTools` (boolean) - Whether to auto-execute tool calls (default: true)
- `cognition` (boolean) - Enables `<think>, <plan>, <speak>` XML tags
- `chainRun` (boolean) - Enables chain running mode with finish_run tool
- `maxChainIterations` (number) - Maximum iterations for chain running (default: 5)
- `structuredOutputSchema` (object) - Schema for structured output (cannot use with tools or cognition)
- `additionalParams` (object) - Extra LLM API parameters (temperature etc.)
- `debug` (boolean) - Enables debug GUI monitoring
- `forceTool` (boolean) - Forces the agent to use exactly one tool (requires exactly one tool in tools array, cannot be used with chainRun)

### MODIFYING AN AGENT'S MESSAGE HISTORY
You can modify an agent's message history with the following methods:

```typescript
// Adding messages
agent.addUserMessage("Hello, how are you? Do you like my hat?", {images: ["https://example.com/blueHat.jpg"]}) // image optional
agent.addAssistantMessage("I am fine, thank you! Nice blue hat! Looks good on you!")

// Loading in custom message history
const history = [{role: "user", content: "Hello, how are you? Do you like my hat?", images: [{url: "https://example.com/blueHat.jpg"}]}, {role: "assistant", content: "I am fine, thank you! Nice blue hat! Looks good on you!"}] // array of messages
agent.loadHistory(history) // loads the chat history from an array of messages

// Extracting current message history
agent.extractHistory() // returns the chat history as an array of messages
```

### COGNITION
Cognition is the process of the agent thinking, planning, and speaking. It is enabled by the cognition property in the agent config. What is does is add forced instructions at the end of the agent's system prompt to use XML tags to think, plan, and speak. These XML tags are parsed and executed by the agent. `<think>...</think>`, `<plan>...</plan>`, `<speak>...</speak>` are the tags used. `<speak>` tags are parsed and returned as the agent's response.

I find that cognition is a great way to get increased accuracy and consistency with tool usage.

### TOOL USE
Tool calls (also known as function calling) allow you to give an LLM access to external tools.

Feather expects your tool to be defined WITH the function execution and output ready to go. By default, tools auto-execute - when giving an agent a tool, the agent will execute the tool, get the results saved in its chat history, then re-run itself to provide the user a detailed response with the information from the tool result.

However, you can disable auto-execution by setting `autoExecuteTools: false` in the agent config. In this case, tool calls will be available in the `functionCalls` property of the response, allowing for manual handling:

```typescript
const manualAgent = new FeatherAgent({
  systemPrompt: "You are a math tutor who can do calculations",
  tools: [calculatorTool],
  forceTool: true, // forces the agent to use this tool instantly and return the result
  autoExecuteTools: false // Disable auto-execution
});


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingbootoshi/feather](https://github.com/kingbootoshi/feather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
