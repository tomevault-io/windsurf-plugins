---
trigger: always_on
description: - **Monitor and Evaluate LLM applications**: Utilize tools like LangSmith for monitoring application performance, including logging traces, analyzing latency, and evaluating LLM outputs against predefined metrics. This helps identify bottlenecks and ensures the application meets quality standards. Enable tracing by setting environment variables:
---

- **Monitor and Evaluate LLM applications**: Utilize tools like LangSmith for monitoring application performance, including logging traces, analyzing latency, and evaluating LLM outputs against predefined metrics. This helps identify bottlenecks and ensures the application meets quality standards. Enable tracing by setting environment variables:
	`export LANGCHAIN_TRACING_V2="true"`
	`export LANGCHAIN_API_KEY="..."`

- **Implement Stateful Agents**: Use LangGraph to build stateful agents, crucial for applications like chatbots where remembering past interactions enhances user experience. Model interactions as a graph with nodes (states) and edges (transitions).

- **Maintain High Code Quality**: Enforce strict code quality through regular testing, ESLint, and Prettier for consistent code formatting and linting.

- **Comprehensive Documentation**: Ensure all components and their interactions are well-documented for maintainability and scalability.

- **Use LangChain Expression Language (LCEL)**: Employ LCEL for composing chains in a declarative way, supporting production deployment without code changes.

- **Explore Trade-offs in Deployment**: Choose between using external LLM providers or self-hosting open-source models based on cost, latency, and privacy considerations.

- **Secure Coding Practices**: Read up on our Security best practices to make sure you're developing safely with LangChain.

## 1. Code Organization and Structure

- **Directory Structure**: Organize code into logical modules based on functionality (e.g., `chains`, `agents`, `tools`, `memory`).
  
  src/
  ├── chains/
  │   ├── conversationalChain.ts
  │   └── summarizationChain.ts
  ├── agents/
  │   ├── agent.ts
  │   └── agentExecutor.ts
  ├── tools/
  │   ├── searchTool.ts
  │   └── calculatorTool.ts
  ├── memory/
  │   ├── bufferMemory.ts
  │   └── conversationBufferMemory.ts
  ├── utils/
  │   ├── api.ts
  │   └── helpers.ts
  ├── index.ts
  └── types.ts
  

- **File Naming Conventions**: Use descriptive names, typically in `camelCase` for variables and functions, and `PascalCase` for classes and interfaces.  Consider prefixes or suffixes to denote the type of module e.g., `*_chain.ts` or `*_agent.ts`

- **Module Organization**: Group related functionalities into modules with clear interfaces. Use `index.ts` files to export module members for cleaner imports.
  typescript
  // chains/index.ts
  export * from './conversationalChain';
  export * from './summarizationChain';
  
  // Usage:
  import { ConversationalChain, SummarizationChain } from '@/chains';
  

- **Component Architecture**: Design modular components for reusability. Implement interfaces to define contracts between components.
  typescript
  // Define an interface for a Tool
  interface ToolInterface {
      name: string;
      description: string;
      execute(input: string): Promise<string>;
  }

  // Implement the interface in a specific Tool
  class SearchTool implements ToolInterface {
      name = 'search';
      description = 'Useful for searching the internet.';
      async execute(input: string): Promise<string> {
          // Implementation
      }
  }
  

- **Code Splitting**: Implement code splitting using dynamic imports to reduce initial load time, especially for large applications.
  typescript
  async function loadLargeModule() {
      const largeModule = await import('./largeModule');
      largeModule.initialize();
  }
  

## 2. Common Patterns and Anti-patterns

- **Design Patterns**: Use the Factory pattern for creating different types of chains or agents, and the Strategy pattern for choosing different LLMs.
  typescript
  // Factory Pattern for creating chains
  class ChainFactory {
      static createChain(type: 'conversational' | 'summarization', llm: LLM) {
          if (type === 'conversational') {
              return new ConversationalChain(llm);
          } else if (type === 'summarization') {
              return new SummarizationChain(llm);
          } else {
              throw new Error('Invalid chain type');
          }
      }
  }

  const chain = ChainFactory.createChain('conversational', new OpenAIChat());
  

- **Recommended Approaches**: Leverage LangChain's built-in modules and chains when possible, and customize them as needed. Prioritize asynchronous operations to prevent blocking the main thread.

- **Anti-patterns**: Avoid deeply nested callbacks, which can lead to callback hell. Use `async/await` and promises for cleaner asynchronous code.

- **State Management**: For simple applications, manage state with React's `useState` or similar. For complex applications, consider state management libraries like Zustand or Redux.

- **Error Handling**: Implement robust error handling with `try/catch` blocks and global error handlers. Log errors with context for debugging.
  typescript
  async function processData() {
      try {
          const result = await fetchData();
          // Process result
      } catch (error) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mariano-a-dim) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
